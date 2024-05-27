<!--yml
category: 未分类
date: 2024-05-27 13:14:19
-->

# Upgrading 3 Million Variables to Envelope Encryption

> 来源：[https://blog.railway.app/p/envelope-encryption](https://blog.railway.app/p/envelope-encryption)

Railway is (temporarily, at least) hosted entirely on Google Cloud Platform while we begin [transitioning our infrastructure to bare metal](https://blog.railway.app/p/team-spotlight-infrastructure-engineering#a-glimpse-into-railway%27s-dynamic-infrastructure-team).

In the beginning, taking advantage of existing GCP-provided services to launch a start-up made total sense. Building core services like container registry, block storage, load balancers, or key management would’ve only gotten in the way of building a product.

Moving to bare metal gets us the margins required to build a profitable business but, even disregarding cost, relying on a single infrastructure  provider is a huge technical risk. “How often does GCP fail though?” You  may ask.  Enough to warrant an entire [blogpost](https://blog.railway.app/p/gcp-incidents) on the subject!

The GCP service that continues to cause the most problems for us is Key Management Service (KMS). And, while the dependency still technically exists, our usage of KMS has been greatly reduced, making migrating away almost trivial.

Let’s dig into how we removed this KMS dependency while also improving performance, all with no downtime.

View of cryptographic requests to KMS before and after

Railway is all about making it easy for users to deploy code. These deployments often require configuration values like API keys and database credentials to be available. Railway solves this with [Service Variables](https://docs.railway.app/reference/variables#service-variables), which are automatically injected into deployments at runtime.

Since these values often contain sensitive information, they get encrypted prior to storage in our Postgres database. Until now, these encryption operations have been done by calling KMS directly.

This works as expected when the number of variables per deployment is average (around 10), but some deployments have hundreds 🫣.

This spiky KMS usage pattern leads to problems.

*   KMS is an external service → Decrypting hundreds of variables for a deployment requires many network request (~20ms each) which slows down large batches, even if done concurrently
*   KMS enforces a quota → Exceeding the quota will cause all operations to fail temporarily. Quotas can be raised by request but approval is not guaranteed, and [GCP has lowered quotas without warning](https://twitter.com/JustJake/status/1667492928758095872?s=20) in the past
*   KMS is not meant for unlimited keys → All variables are currently encrypted using the same KMS key, meaning a brute-force attack could compromise variables across all of Railway

Fortunately, there’s a standard practice that solves all these problems at once and is actually what KMS is designed for. (Yes, we were holding it wrong.)

Envelope encryption is the practice of encrypting plaintext data — like variables — with a data encryption key (DEK), then encrypting the DEK with a key encryption key (KEK).

This reduces the surface area of an attack by allowing each user (or Railway project) to use separate encryption keys. Using locally generated data keys also means less reliance on an external key management service. Precisely what we’re after!

As mentioned, KMS is designed for envelope encryption and is the reason for its maximum input size of  `64KiB` (enough to encrypt a large DEK). We knew this when implementing variable encryption, but since 99% of variables fit within the maximum size (about the length of a 10,000 word blogpost), it made sense to use KMS directly to get feature out the door.

Envelope encryption eliminates the need call KMS hundreds of times for large batches of variables by requiring only one data encryption key (DEK) for the entire batch. We simply generate one DEK per [Environment](https://docs.railway.app/guides/environments) (an isolated instance of a project, like production or staging) and use it for all encryption operations.

The DEK is generated locally (256-bit [AES using](https://en.wikipedia.org/wiki/Galois/Counter_Mode) [GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode)), encrypted once using KMS, then stored on the `Environment` data model. We created a helper utility to handle encryption, decryption, and key generation, and attached it to our request context, which has a lifetime of the current HTTP request or background job.

```
interface EnvelopeEncryptionManager {
  generateEncryptedDek(): Promise<string>;
  encrypt(plaintext: string, encryptedDek: string): Promise<string>;
  decrypt(encrypted: string): Promise<string>;
}
```

This means that for each deployment (or other action), the encryption helper decrypts the necessary DEK once and caches it in memory for further operations, reducing  the 100+ trips to KMS to 1.

There’s some other magic in here though. Did you notice that `decrypt()` doesn’t require a DEK? How does that work?

One of the requirements for envelope encryption was the ability to easily rotate keys. As we’ve seen, the DEK is stored on the `Environment` model, but how would we rotate it if needed? We’d either have to store a list of old keys to try during the encryption process, or re-encrypt all variables and update the database inside a potentially huge database transaction.

Not great.

Luckily there’s also a best-practice for this, which is to store the encryption key next to the encrypted value (cipher text). For us, this meant serializing the encrypted DEK with the cipher text before storing the value in the database as a JSON string:

```
variable.encryptedValue = `{"encryptedDek":"...","cipherText":"..."}`
```

Variable decryption with this setup can be done by decrypting the DEK using KMS (if not already cached in memory) and using it to decrypt the variable locally (without KMS). It also makes passing around encrypted values easier because the associated `Environment` or DEK doesn’t need to go along for the ride.

The basic implementation of this strategy was only a few hundred lines of code but we still had to convert over 3 million existing variables to the new system.

Storing the encrypted DEK next to the cipher text is also what made it possible to support both legacy and envelope encryption simultaneously, allowing us to progressively upgrade over 3 million variables.

For decryption fallback, if a DEK isn’t found within the encrypted variable, KMS is called directly to decrypt just like before.

Similarly for encryption fallback, if no DEK exists yet on the `Environment`, KMS is called directly.

This makes it possible to generate a DEK for a single environment. Modified variables within this environment will be upgraded to envelope encryption on write, reducing the migration effort to the following steps:

1.  Deploy initial envelope encryption code (does nothing by itself)
2.  Start generating DEKs for newly created environments
3.  Run data migration to generate DEKs for all environments
4.  Run data migration to re-encrypt all variables

It’s been a few weeks since introducing envelope encryption and it’s been a success. Our KMS usage has dropped by 10x with no usage spikes, and we’ve been able to realize the benefits mentioned earlier:

*   Better performance → Decrypting DEKs with KMS once per batch reduced the slow-case for variable decryption from ~10s at the slowest to only a handful of milliseconds
*   No service disruption → The more consistent usage pattern means we no longer have to worry about spiking above our KMS quota and causing downtime. We’ve also been able to reduce our quota by several orders of magnitude
*   Better security → Using a separate data key per environment means that a brute-force attack of one environment won’t compromise data for any others
*   More flexible encryption → While we don’t necessarily need to surpass the `64KiB` KMS limit, the use of symmetric AES encryption means we can now encrypt data of any length

Not only do we get these direct benefits but we’ve also already used this envelope encryption system to persist credentials for our recently-released [Private Registry Support](https://railway.app/changelog/2024-03-15-template-composer).

We’re one step closer to having no dependencies on GCP. Yes, we’re still technically using KMS, but replacing it with something like [Vault](https://www.vaultproject.io/) can now be done incrementally by repeating the above migration steps to rotate the DEKs for each environment.

So that’s how we upgraded 3 million variables to envelope encryption. Migrating data is always a scary task. Migrating sensitive data is even worse! But, as it goes, even the most daunting work can be broken down into manageable chunks, which is how we lived to tell this tale.

Did we do something wrong? Let us know on [Twitter](https://twitter.com/railway) and stay tuned for more of these stories as we get further and further along our path to bare metal.