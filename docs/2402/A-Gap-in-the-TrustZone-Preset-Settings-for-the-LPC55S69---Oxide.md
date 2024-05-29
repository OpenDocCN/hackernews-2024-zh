<!--yml
category: 未分类
date: 2024-05-27 14:40:58
-->

# A Gap in the TrustZone Preset Settings for the LPC55S69 / Oxide

> 来源：[https://oxide.computer/blog/lpc55s69-tzpreset](https://oxide.computer/blog/lpc55s69-tzpreset)

The LPC55S69 offers debug access via [standard ARM interfaces (SWD)](https://developer.arm.com/documentation/ihi0031/g/?lang=en). Debug access can be configured to be always available, always disabled, or only available to authenticated users. These settings are designed to be applied at manufacturing time via the [CMPA](https://github.com/nxp-mcuxpresso/spsdk/blob/09c711a8fd4c54f126a7dfe1b3ae8bb361c5473e/spsdk/data/pfr/cmpa/lpc55s6x_1b.xml) region. Debugging is disabled by default while executing in the ROM and only enabled (if allowed) as the very last step before jumping to user code. The debug settings are also locked out, preventing further modification from user code except in specific authenticated circumstances. Because debug access is highly sensitive, it makes sense to minimize the amount of time the ROM spends with it enabled.

If the debug settings are applied last, this means that the TrustZone preset settings must be applied before them. Combine this information with the implementation detail of how the preset setting are applied, **if the code faults after `VTOR` is changed but before we apply the debug settings, it will be possible to run in user controlled code with debug registers open for modification**.

How easy is it to actually trigger this? Very easy. Other registers in the preset structure include settings for the MPU. Setting the enable bit in `MPU_CTRL` without any other regions set is enough to trigger the fault. NXP actually says in their manual that you need to make sure the entire ROM region is configured as secure privileged and executable otherwise "boot process will fail". "fail" in this case is vectoring off into the appropriate fault handler of the user code.

This makes the following sequence possible:

*   Have debug disabled in the CMPA

*   Sign an image with TrustZone preset settings with a valid `VTOR` and MPU settings that exclude the ROM region

*   Have the `MemManage` fault handler follow the standard sequence to enable debugging

*   The image will trigger the fault handler and have debugging enabled despite the settings in the CMPA

This does require access to the secure boot signing key, but it’s a departure from the presentation of the CMPA settings as being independent of any possible settings in an image.