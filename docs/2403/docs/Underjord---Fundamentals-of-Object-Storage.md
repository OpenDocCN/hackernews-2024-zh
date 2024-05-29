<!--yml
category: 未分类
date: 2024-05-29 12:36:56
-->

# Underjord | Fundamentals of Object Storage

> 来源：[https://underjord.io/fundamentals-of-object-storage.html](https://underjord.io/fundamentals-of-object-storage.html)

# Fundamentals of Object Storage

2024-03-19

Underjord is a [tiny, wholesome team](/team.html) doing Elixir consulting and contract work. If you like the writing you should really try the code. See our [services](/services.html) for more information.

I did a livestream where I talked about Object Storage. The how and the why. The Bad Old Days. And also the neat and interesting stuff just beyond the basics. I figured I’d cover that in text as well.

You can [watch the stream here](https://youtube.com/live/Sg0Afr_4BMY).

*Transparency notice: The livestream and this blog post have been work supported by [Tigris](https://www.tigrisdata.com/). We collaborate on the topics, they provide resources and pay for some of my time so I can do this stuff instead of writing software for clients. I am very glad and thankful that I have someone funding my publishing and creative ideas. Expect more output from me near term, and give them a try!*

## Background

When I cut my teeth architecting systems we didn’t have Objects outside of Object-Oriented programming languages and we had precious little Storage. And if we did have Storage it was either on the same physical server or some kind of NFS-based abomination.

Imagine you set up your n-tier architecture. Application scales horizontally in front of the database. You can cache heavy loads with some Redis or Memcache that were the rad things then. For file storage we had to set up a shared store of some sort. And they were finicky bastards.

The mounts would come loose and files got written in a local folder. The performance would randomly degrade. I remember installing a filesystem cache thing for my servers and with NFS it would suddenly just hard-lock the system at the kernel level. Because NFS lives or at least lived then, in the kernel.

When it works well it can be very practical to have a networked filesystem pretend to be a real one. But it is a pretty little lie that you are telling yourself and the system you are building. And if you lean into the file-ish nature of the NFS lifestyle … well, there are risks. And scars you can get.

I found AWS annoying when it arrived. Unreliable VPS:es that you weren’t supposed to put files on. Bah. Humbug. So it took a while before I got into Object Storage (all object storage is essentially S3-compatible these days). But much like the dumbness of EC2-instances enforced good scalability practices the simplicity of S3 made it tremendously effective.

What people want from a network file store is usually reliability and space. The data stays where you put it and does not get lost. And your data must fit. And as data grows it must still fit. So essentially infinite unknown unbounded storage growth. And ideally you don’t want to pay more than disk utilization you currently need.

Simplicity, restraint and constraints are good for starting most things. But especially to delivering on ambitious things. Like infinite* file storage.

* not actually infinite, but for most purposes close enough

Object Storage is much more clearly a service. It is not a file system. And when you scratch the surface the name clarifies itself a bit. It is not necessarily about “files” either. It is just the most successful NoSQL DB of all time probably. Keys, values and it doesn’t sweat the rest so much.

The fundamental operations are:

*   Put object
*   Get object
*   List objects
*   Delete object

The full list is [much, much longer](https://docs.aws.amazon.com/AmazonS3/latest/API/API_Operations_Amazon_Simple_Storage_Service.html).

## With Elixir

To show how we can work with Object Storage fundamentals in Elixir I set up a mix project: `mix new bla`. Then I added the following deps in `mix.exs`:

elixir mix.exs

```
 defp deps do
    [
      {:ex_aws, "~> 2.5"},
      {:ex_aws_s3, "~> 2.5"},
      {:hackney, "~> 1.9"},
      {:sweet_xml, "~> 0.7"},
      {:jason, "~> 1.4"}
    ]
  end
```

I then created a `config/config.exs` with the following contents:

elixir config/config.exs

```
import Config

config :ex_aws, :s3,
  scheme: "https://",
  host: "fly.storage.tigris.dev",
  port: 443
```

To get your deps: `mix deps.get`

To create a bucket, if you have the `fly` command-line tool ready to go it is very simple to do `fly tigris create`. It is also free until your **really** use it, 5Gb and 10k+ requests/month. Of course take whatever bucket you prefer. But that will spit out credentials.

`:ex_aws_s3` will automatically slurp up your credentials if you set them as environment variables but not the bucket name. If you need multiple buckets you can wrangle them explicitly. These examples assume one set of credentials.

I started the module I called `Tigris` like this:

elixir lib/tigris.ex

```
defmodule Tigris do
  alias ExAws.S3

  defp bucket!, do: System.fetch_env!("BUCKET_NAME")
end
```

Then I tackled the listing of objects:

elixir lib/tigris.ex

```
#..
  def list!(prefix \\ "") do
    bucket!()
    |> S3.list_objects(prefix: prefix)
    |> ExAws.request!()
    |> then(fn %{body: %{contents: contents}} ->
      contents
    end)
  end
#..
```

You can try things in `iex -S mix` to get everything we have in this project compiled and ready to go in the prompt:

elixir iex

```
iex(1)> Tigris.list!()
[]
```

Next some putting and getting. I am not suggesting you do error handling like this. This is for the purposes of simplicity and the stream, not a best practice and definitely not financial or legal advice.

elixir lib/tigris.ex

```
# ..
  def put!(key, data) do
    bucket!()
    |> S3.put_object(key, data)
    |> ExAws.request!()

    :ok
  end

  def get(key) do
    result =
      bucket!()
      |> S3.get_object(key)
      |> ExAws.request()

    case result do
      {:ok, %{body: body}} -> body
      {:error, {:http_error, 404, _}} -> nil
      {:error, error} -> {:error, error}
    end
  end
# ..
```

I tried those out in `iex` as well to show the fundamentals. Then I wanted to hint at scale I guess. `Task.async_stream/2` ensures things are run in plenty parallel according to your machine.

elixir lib/tigris.ex

```
# ..
  def put_tons!(kv) do
    kv
    |> Task.async_stream(fn {key, value} ->
      IO.puts(key)
      put!(key, value)
    end)
    |> Stream.run()
  end
# ..
```

And then:

elixir iex

```
iex(1)> kv = for d <- 1..100, f <- 1..100, into: %{}, do: {"dir-#{d}/file-#{f}.txt", "#{d}  #{f}"}
# snip
iex(2)> Tigris.put_tons!(kv)
```

And then to tidy up. Don’t run this on buckets where you keep data you like or need:

elixir lib/tigris.ex

```
# ..
  def exterminate! do
    stream =
      bucket!()
      |> S3.list_objects()
      |> ExAws.stream!()
      |> Stream.map(& &1.key)

    S3.delete_all_objects(bucket!(), stream) |> ExAws.request()

    :exterminated
  end
# ..
```

## Presigned URLs

No we are getting a bit fancy. So an enormously beneficial thing with Object Storage is that it can have features that spare your application server from a bunch of horrible work. Have you ever let your app receive a file upload only to save it somewhere else? Ludicrous! We have a service for that! There are protocols!

elixir lib/tigris.ex

```
# ..
  def presign_get(key) do
    :s3
    |> ExAws.Config.new([])
    |> S3.presigned_url(:get, bucket!(), key, [])
  end
# ..
```

Use that. Get a presigned URL (you can configure expiration and such) for either downloading or uploading which you can hand off to the client and let them deal with the upload without you. Just between them and your object storage. Fun thing about doing that with Tigris btw. Tigris will place the data globally close to the uploader. It will then replicate as a cache if needed to other regions but this means your customers in Australia get local latencies.

This is equally nice for uploads and downloads. Your application server does not have to go between. At worst a download is:

*   Client asks your application server for a file.
*   Your application checks if the request makes sense and should be signed for.
*   Presign URL.
*   Client receives a response with a redirect to the presigned URL.
*   Transparent download but your application doesn’t serve the bytes.

## Multipart upload (streaming up!)

Up to 5Gb can be a single upload according to AWS. But typically that gets unwieldy. We can chunk uploads at 5Mb chunks.

elixir lib/tigris.ex

```
# ..
  def put_file!(key, from_filepath) do
    from_filepath
    |> S3.Upload.stream_file()
    |> Stream.map(fn chunk ->
      IO.puts("uploading...")
      chunk
    end)
    |> S3.upload(bucket!(), key)
    |> ExAws.request!()
  end
# ..
```

This is a form of streaming upload. Which means if you need to do processing and then want to offload the result immediately you can reduce your memory usage to about 5Mb (it depends) by streaming the upload instead of holding on to the whole beastly thing. This specifically builds a Stream from a file for upload but there are many variants you can do using the Elixir Stream and IO tools.

## Range requests (streaming down!)

Sometimes we just want a few parts of a file. Sometimes we want to build a hell-beast that does read-only SQLite VFS over S3 API. Sometimes we want to do a graceful streaming download. For this we have straight up HTTP range requests.

elixir lib/tigris.ex

```
# ..
  def get(key, range \\ nil) do
    opts =
      if range do
        [range: "bytes=#{range}"]
      else
        []
      end

    result =
      bucket!()
      |> S3.get_object(key, opts)
      |> ExAws.request()

    case result do
      {:ok, %{body: body}} -> body
      {:error, {:http_error, 404, _}} -> nil
      {:error, error} -> {:error, error}
    end
  end
# ..
```

This is incredibly useful and powerful. Sure. You might mostly be dealing with files. Until you don’t. Or until you realize how many file formats expose useful information with just the right bytes.

Zip archives are streamable with [Packmatic](https://github.com/evadne/packmatic) because they allow you to keep the index of files to decompress at the end of the archive. That’s a known location. We can search the last part of the file and only get the index, and so, the file listing. And that means we can pick out files. One can do similar things with ID3 tags on MP3s and get metadata without getting the whole file.

If you use the Elixir library [Explorer](https://hex.pm/packages/explorer) the underlying Rust library supports the S3 API and will let you lazily read data from .parquet files and such that are stored remotely. Same thing.

I hope this clarifies why range requests are rather useful for a file storage service.

## Why the S3 API became standard

All Object Storage services I’ve used so far have exposed an S3-compatible API. It makes sense. It helps you support the expected featureset and you get a trillion clients and SDKs compatible with your thing for free.

Is the S3 API just that good?

I don’t think it is. And I had a whole spiel here about how it is simply good enough and that the value proposition of reliable “infinite” storage with a common protocol makes it worth it. It clearly has been. In discussing this post with Ovais Tariq, CEO of Tigris, he actually shared a much more interesting viewpoint. Contrarian to my lukewarm take. It makes sense for Tigris to ship as S3 API compatible to make switching and getting started simple. The S3 API turns out to be a significant constraint however. Tigris has a metadata system and infrastructure that is different from S3\. There are improvements, innovations and some really great features that are not easy to implement nicely in the fairly stagnant S3 API. Sure you can finagle features into headers and other clever stuff but there is a downside to it as any new ground you break will be missing in client implementations. From my conversations with Ovais and his team we will still see innovations on Object Storage from them. I guess we’ll have to stay tuned to see if the S3 API can handle it.

* * *

If you want to share some of the more interesting usages you’ve seen of Object Storage or just want to tell me about whether you found this helpful, feel free to reach out. I am on the Fediverse as [@lawik](https://twitter.com/lawik) and you can just email me as well at [lars@underjord.io](mailto:lars@underjord.io). Thanks for reading.

Underjord is a [4 people team](/team.html) doing Elixir consulting and contract work. If you like the writing you should really try the code. See our [services](/services.html) for more information.

Note: Or try the videos on [the YouTube channel](https://youtube.com/c/underjord).