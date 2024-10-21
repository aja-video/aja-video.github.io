# NTV2 Porting Guide

This article suggests how to port application code that builds against an older NTV2 SDK to the most recent SDK.

_(Last updated:  October, 2024)_

## Porting from the pre-13.0 “ancient world”

These four steps will cover 80% of the porting effort:
  1. [Port “NTV2RoutingEntry” to use “NTV2InputCrosspointID”s](ntv2-porting-guide-ntv2routingentry.md)
  2. [Port “CNTV2Task” to new “AutoCirculateTransfer” API](ntv2-porting-guide-ntv2task.md)
  3. Replace old AutoCirculate calls with the new ones:
  - Replace all calls to `InitAutoCirculate` with `AutoCirculateInitForInput` or `AutoCirculateInitForOutput`
  - Replace all calls to `StartAutoCirculate` with `AutoCirculateStart`
  - Replace all calls to `StopAutoCirculate` with `AutoCirculateStop`
  - Replace all calls to `AbortAutoCirculate` with `AutoCirculateStop(channel, true)`
  - Replace all calls to `PauseAutoCirculate` with `AutoCirculatePause` or `AutoCirculateResume`
  - Replace all calls to `TransferWithAutoCirculate` with `AutoCirculateTransfer`
  - Replace all calls to `FlushAutoCirculate` with `AutoCirculateFlush`
  - Replace all calls to `PrerollAutoCirculate` with `AutoCirculatePreRoll`
  - Replace all calls to `GetAutoCirculate` with `AutoCirculateGetStatus`
  - Replace all calls to `GetFrameStamp` with `AutoCirculateGetFrame`
  4. Heed all compile-time deprecation warnings (e.g., “‘XXX’ is deprecated, use ‘YYY’ instead”).

If it compiles and links without error, _congratulations!_ — your application is compliant with the latest SDK.

If not, make note of the symbols in the compilation errors. This article will show you how to fix them.

> **NOTE:** You can always try to build with a new SDK, but turn off (comment-out) the `#define NTV2_DEPRECATE…` macros in `ajatypes.h` which restores most of the old symbols and APIs — but this is not a permanent solution. 

### SDK Source Layout Changes

SDK version 13.0 was the first to have the source directories match on all three platforms (Linux, MacOS and Windows). In addition, the SDK was shipped as a **.zip** file on Windows and Linux for the first time.
- Projects that referred to `ntv2projects/classes` now should refer to `ajalibraries/ajantv2/src`.
- Projects that referred to `ntv2projects/includes` now should refer to `ajalibraries/ajantv2/includes`.
- Projects that referred to `ajastuff/includes` now should refer to `ajalibraries/ajabase/includes`.
- Replace any occurrences of `#include "ajastuff/` with `#include "ajabase/`.

#### Obsolete/Superceded Source Files

The contents of a number of source and header files have become obsolete and/or assumed into other source/header files. The obsolete files may be removed in a future SDK.

Obsolete/superceded implementation files:
- `ntv2colorcorrection.cpp`
- `ntv2procamp.cpp`
- `ntv2status.cpp`
- `ntv2testpattern.cpp`
- `xena2vidproc.cpp`

Obsolete/superceded headers:
- `ntv2boardscan.h` -- Replace all occurrences of `#include "ntv2boardscan.h"` with `#include "ntv2devicescanner.h"`
- `ntv2boardfeatures.h` -- Replace all occurrences of `#include "ntv2boardfeatures.h"` with `#include "ntv2devicefeatures.h"`
- `ntv2choosableboard.h`
- `ntv2colorcorrection.h` -- Replace all occurrences of `#include "ntv2colorcorrection.h"` with `#include "ntv2card.h"`
- `ntv2procamp.h` -- Replace all occurrences of `#include "ntv2procamp.h"` with `#include "ntv2card.h"`
- `ntv2status.h` -- Replace all occurrences of `#include "ntv2status.h"` with `#include "ntv2card.h"`
- `ntv2testpattern.h` -- Replace all occurrences of `#include "ntv2testpattern.h"` with `#include "ntv2card.h"`
- `ntv2vidproc.h` -- Replace all occurrences of `#include "ntv2vidproc.h"` with `#include "ntv2card.h"`
- `ntv2vidprocmasks.h`
- `resample.h` -- Replace all occurrences of `#include "resample.h"` with `#include "ntv2resample.h"`
- `testpatterndata.h`
- `testpatterngendata.h`
- `transcode.h` -- Replace all occurrences of `#include "transcode.h"` with `#include "ntv2transcode.h"`
- `verticalfilter.h` -- Replace all occurrences of `#include "verticalfilter.h"` with `#include "ntv2verticalfilter.h"`
- `xena2routing.h` -- Replace all occurrences of `#include "xena2routing.h"` with `#include "ntv2signalrouter.h"`
- `xena2vidproc.h` -- Replace all occurrences of `#include "xena2vidproc.h"` with `#include "ntv2card.h"`
- `xenaserialcontrol.h` -- Replace all occurrences of `#include "xenaserialcontrol.h"` with `#include "ntv2serialcontrol.h"`

## API Changes

It’s a good practice to start at the Knowledgebase article for the SDK version your application is currently built for, then “walk” forward through the Knowledgebase to each successive SDK, noting the API Changes. This section condenses many of these changes into a series of search/replace steps, for your convenience.

1. Replace all occurrences of `NTV2_TCDEST_…` with `NTV2_TCINDEX_…`
1. Replace all occurrences of `NTV2_TCSOURCE_…` with `NTV2_TCINDEX_…`
1. Replace all occurrences of `NTV2DeviceGetNumAudioStreams` with `NTV2DeviceGetNumAudioSystems`
1. Replace all uses of `NTV2DeviceCanDoLTCOutN(id,N)` with `NTV2DeviceGetNumLTCOutputs(id) > N`
1. Replace all occurrences of `NTV2BoardCanDo…` with `NTV2DeviceCanDo…`
1. Replace all occurrences of `NTV2BoardGetNum…` with `NTV2DeviceGetNum…`
1. Remove all occurrences of `NTV2BoardType`.
1. Remove all uses of `NTV2DeviceType`, `DEVICETYPE_UNKNOWN` and/or `DEVICETYPE_NTV2`.
1. Any/all `CNTV2Card::Open` calls should have at most two parameters: the index number and an optional host name (now a `std::string`).
1. Remove all calls to the `NTV2DeviceTypeString` function.
1. Remove all uses of `BOARDTYPE_SCANNABLE`.
1. Remove all uses of `BOARDTYPE_NTV2`.
1. Remove all uses of `BOARDTYPE_AS_COMPILED`.
1. Replace all occurrences of `BOARD_ID_…` with `DEVICE_ID_…`.
1. Remove all uses of `NTV2BoardSubType`.
1. Remove all uses of `BOARDSUBTYPE_…`.
1. Replace all occurrences of `NTV2_V2_STANDARD_…` with `NTV2_STANDARD_…`.
1. Replace all occurrences of `NTV2V2Standard` with `NTV2Standard`.
1. Replace all occurrences of `IS_PROGRESSIVE_NTV2Standard` and `IS_PROGRESSIVE_STANDARD` with `NTV2_IS_PROGRESSIVE_STANDARD`.
1. Replace all occurrences of `NTV2_IS_VALID_NTV2Standard` and `NTV2_IS_VALID_NTV2V2Standard` with `NTV2_IS_VALID_STANDARD`.
1. Anywhere an `NTV2FrameDimensions` value was used, replace all occurrences of `.cx` or `.cy` with `.Width()` and `.Height()` respectively.
1. Replace calls to `IsKonaIPDevice` with `IsIPDevice`.
1. Replace all occurrences of `AJATestPatternGen` with `NTV2TestPatternGen`.
1. Replace calls to `getTestPatternList` with calls to `getTestPatternNames`.
1. Replace all uses of `AJATestPattern…` data types with `NTV2TestPattern…` ones.
1. Replace all uses of `AJATestPattEx…` enums with `NTV2_TestPatt_…` ones.
1. Un-numbered `HDMIIn` output crosspoint IDs are now numbered, so replace `NTV2_XptHDMIIn…` with e.g. `NTV2_XptHDMIIn1`.

### API Changes — NTV2FormatDescriptor and NTV2VANCMode:

1. Any source file that contains `NTV2FormatDescriptor` must `#include "ntv2formatdescriptor.h"` (or include another header file that includes it).
1. Any use of two booleans for “_vancEnabled_” and “_wideVanc_” (or “_tallerVanc_”) should simply use a single `NTV2VANCMode`.
1. Replace calls to `GetFormatDescriptor` with the equivalent `NTV2FormatDescriptor` constructor.
1. Replace calls to `GetVideoWriteSize (videoFormat, pixelFormat, isVancEnabled, isWideVanc)` with `GetVideoWriteSize(videoFormat, pixelFormat, vancMode)`
1. Replace calls to `SetEnableVANCData` with `SetVANCMode`.
1. Replace calls to `GetEnableVANCData (&isVancEnabled, &isTallVanc, channel)` with `GetVANCMode (vancMode, channel)`

### API Changes — AJA Caption Library

1. Replace all calls to `IsCaptionChannel` with call to `IsLine21CaptionChannel`
1. Replace all calls to `IsTextChannel` with call to `IsLine21TextChannel`
1. In **SDK 14.2**, caption library logging changed substantially to use the vastly more convenient AJA Logger utility.
  1. Most of the `kCaptionLog_` constants were removed.
  1. `GetDefaultCaptionLogOutputStream` and `SetDefaultCaptionLogOutputStream` were removed.
  1. The `Log`, `LogIf` and `SetLogStream` methods are deprecated. Don’t use them.

### API Changes — CXena2VidProc (Mixer)

1. Replace all uses of `CXena2VidProc` class with `CNTV2Card`.
1. Replace all calls to `GetXena2VidProcMode`, `GetXena2VidProc2Mode`, `GetXena2VidProc3Mode`, etc. with `GetMixerMode`, specifying the Mixer index number 0, 1, 2, etc. as appropriate.
1. Replace all calls to `SetXena2VidProcMode`, `SetXena2VidProc2Mode`, `SetXena2VidProc3Mode`, etc. with `SetMixerMode`, specifying the Mixer index number 0, 1, 2, etc. as appropriate.
1. Replace all calls to `GetMixCoefficient`, `GetMix2Coefficient`, `GetMix3Coefficient`, etc. with `GetMixerCoefficient`, specifying the Mixer index number 0, 1, 2, etc. as appropriate.
1. Replace all calls to `SetMixCoefficient`, `SetMix2Coefficient`, `SetMix3Coefficient`, etc. with `SetMixerCoefficient`, specifying the Mixer index number 0, 1, 2, etc. as appropriate.
1. Replace all calls to `GetXena2VidProcInputControl`, `GetXena2VidProc2InputControl`, `GetXena2VidProc3InputControl`, etc. with `GetMixerFGInputControl` for `NTV2_CHANNEL1` or `GetMixerBGInputControl` for `NTV2_CHANNEL2`, specifying the Mixer index number 0, 1, 2, etc. as appropriate.
1. Replace all calls to `SetXena2VidProcInputControl`, `SetXena2VidProc2InputControl`, `SetXena2VidProc3InputControl`, etc. with `SetMixerFGInputControl` for NTV2_CHANNEL1 or `SetMixerBGInputControl` for NTV2_CHANNEL2, specifying the Mixer index number 0, 1, 2, etc. as appropriate.
1. Remove all calls to `GetSplitMode`, `SetSplitMode`, and `SetSplitParameters` -- this feature is no longer supported by AJA devices.

### API Changes — Use non-const-references instead of pointers for CNTV2Card “getter” methods:

When a “getter” method has a non-constant-reference return argument, we recommend using that instead of pointers --- e.g., replace `GetXXXX(&variable)` with `GetXXXX(variable)`.

## SDK 12.4.2 changes (from 12.3)

- The SDK now produces compile-time warnings when deprecated APIs are used (on most compilers).
- `CNTV2Task`-based timecode capture/playout (see `AutoCircTimeCodeTask`) is no longer supported in 12.4 (or later) drivers. The `CNTV2Task` and `AUTOCIRCULATE_TASK_STRUCT` API was hacked into the SDK many years ago to allow AutoCirculate clients to frame-accurately set/get timecode to/from SDI spigots other than the channel being circulated. This added a great deal of complexity to the driver and was impossible to scale in order to fix the absence of embedded Field2 VITC in interlaced video. Given our limited resources, we had to simplify the kernel driver and didn’t have sufficient time to back-port the old task-based behavior. We apologize for the inconvenience. If you must frame-accurately set/get timecode to/from SDI spigots that don’t correspond to the AutoCirculate channel, you MUST use the new `AUTOCIRCULATE_TRANSFER` class and `AutoCirculateTransfer` function. A porting guide is [here](ntv2-porting-guide-ntv2task.md).
- SDI input signal monitoring (e.g., VPID valid detect, RX lock tally, TRS error detect, CRC error tally, etc.) on select hardware (e.g. Corvid44) is now implemented on all platforms.
- **New device support:**
  - **Corvid HEVC** — On Windows, support is built into the driver and SDK. On Linux, use the HEVC-specific SDK.
  - **Kona IP** — AJA’s first foray into SMPTE 2022 (video-over-IP)
- **API Changes:**
  - `CNTV2ColorCorrection`, `CNTV2TestPattern`, `CNTV2VidProc`, `CXena2VidProc`, and `CNTV2ProcAmp` have all been deprecated, and their residual functions and member data has been folded into `CNTV2Card`. Henceforth, `CNTV2Card` is now the only class to be used to talk to an AJA device.
  - New `NTV2DeviceCanDoPCMDetection` function and “per-audio-channel-pair” PCM detection member functions of `CNTV2Card` now operate with some newer AJA devices (Corvid44, Kona4 … with newer firmware installed).
  - APIs are now ready to support 128 audio channels (i.e., up to 64 audio channel pairs) per SDI spigot (awaiting future firmware support).
  - `CNTV2AncPacket` and `CNTV2AncPackets` classes were finally removed from the ‘**ajacc**’ library. The `NTV2CCPlayer` demo now uses the ‘**ajaanc**’ library.

## SDK 12.5.0 changes (from 12.4.2)

- **HDR support**
- **SMPTE 2022-7 support on KONA IP**
- **VPID** — major driver improvements to transmit correct VPID, especially for SMPTE 425 (Tsi).
- **64-bit-only on macOS** — The libraries supplied with the SDK now only contain the **x86_64** architecture (not both **i386** and **x86_64**, as before). If a 32-bit executable is required, ‘**classes**’ (and possibly ‘**ajastuff**’, if needed) must be recompiled for **i386**.
- On macOS, the supplied ‘**classes**’ and ‘**ajastuff**’ libraries don’t support the new C++11 LLVM standard library. If you need to link with C++11 support, you must rebuild the libraries.
- **API Changes:**
  - API changes are now tracked using separate per-release macros. `NTV2_DEPRECATE_12_5` is defined in `ajatypes.h` for this release (and can be undefined there, if necessary, to avoid having to make any code changes to accommodate API changes).
  - `NTV2_DEPRECATE` is now defined by default (in `ajatypes.h`). The old, deprecated APIs will be disappearing!
  - **Enhanced:**
    - `NTV2_POINTER`:  New `CopyFrom`, `SwapWith`, and `GetRingChangedByteRange` methods. `IsContentEqual` has been enhanced with new byte offset & count parameters.
    - `AUTOCIRCULATE_STATUS`:  New inquiry methods: `GetAudioSystem`, `IsStopped`, `WithAudio`, `IsInput`, `IsOutput` and `GetChannel`.
    - `CNTV2Card::AutoCirculateGetFrameStamp` — now returns full complement of captured timecodes.
    - `CNTV2Card::WaitForOutputVerticalInterrupt` and `CNTV2Card::WaitForInputVerticalInterrupt` — now have an optional repeatCount parameter (defaults to 1).
    - `CNTV2Card::BankSelectReadRegisters` — has been replaced by `BankSelectReadRegister`.
    - `CNTV2Card::BankSelectWriteRegisters` — has been replaced by `BankSelectWriteRegister`.
    - `NTV2RegInfo` — added several new convenience methods.
    - `NTV2FormatDescriptor` — added several new constructors, plus `GetFirstChangedRow`, `GetChangedLines`, and other convenience methods.
    - `NTV2SMPTELineNumber` — added several new convenience methods.
    - `HevcCommand` & `HevcDeviceCommand` — several new “change encode parameter” options, “change picture type”, etc.
  - **Deprecated:**
    - `NTV2V2Standard` (rolled into existing `NTV2Standard`)
    - `NTV2RoutingEntry` —  This will likely require simple code changes.   A handy porting guide is [here](ntv2-porting-guide-ntv2routingentry.md).
    - `CNTV2Card::DownloadLUTToHW`
    - `NTV2BankRegPair` — no longer needed.
  - **New:**
    - `NTV2VANCMode` — enum to permit deprecation of APIs using the “tall” and “taller” booleans.
    - `NTV2DoubleArray` — for greater ease in getting/setting LUTs.
    - `NTV2RegisterExpert` — one-stop-shopping for register info.
    - `CNTV2Card::LoadLUTTables` and `CNTV2Card::GetLUTTables` methods — which accept `NTV2DoubleArray`s.
    - `CNTV2Card::GetAESOutputSource` and `CNTV2Card::SetAESOutputSource` methods — missing from the API until now.
    - `CNTV2Card::IsAudioChannelPairPresent` and `CNTV2Card::GetDetectedAudioChannelPairs` methods — missing from the API until now.
    - `CNTV2Card::GetDieTemperature` — now you can read the FPGA die temperature in any `NTV2DieTempScale` you like!
    - `AJAAncillaryList::AddVANCData` — for “tall” and “taller” VANC geometries in 10-bit YCbCr frame buffers.
    - `NTV2RasterLineOffsets` — an ordered sequence of zero-based line offsets into a frame buffer.
    - `NTV2InputSourceSet` — a set of distinct `NTV2InputSource` values.
  - **Driver Changes:**
    - New `FRAME_STAMP` message — to see all timecodes being carried in any AutoCirculate frame.
  - **New Device Support:**
    - **Corvid HB-R** — HDBaseT ingest board with VISCA/RS-232 control, 10W of power, captures up to 4K/60p video and 8-channels of audio over Cat5e/6

## SDK 13.0 changes (from 12.5._x_)

- **JPEG-2000 (J2K) support on KONA IP**
- **NEW SDK LAYOUT:**
  - **ajaapps** — Application sources
    - **crossplatform** — Sources for platform-independent applications
      - **demoapps** — Sources for the demonstration apps
  - **ajalibraries** — Sources for all SDK libraries
    - **ajaanc** — The AJA Ancillary Data library
    - **ajabase** — The AJA “Base” library (formerly “ajastuff”)
    - **ajantv2** — The NTV2 library (formerly “ntv2projects” and “classes”)
    - **gpustuff** — OpenGL and other GPU support (Linux and Windows only)
- **API Changes:**
  - `NTV2FormatDescriptor` — Added support for some planar pixel formats.
  - `CNTV2Card` class
    - New `SetAudio20BitMode` and `GetAudio20BitMode` functions — enables or disables 20-bit mode for a given audio engine’s embedder/de-embedder (disabled is the normal 24-bit mode).
    - New `GetAudioOutputEmbedderState` and `SetAudioOutputEmbedderState` functions — to enable or disable an SDI output’s audio embedder.
    - New optional “_inClearDropCount_” parameter in `AutoCirculateResume` and `AutoCirculateFlush` functions — to optionally clear the “dropped frames” tally for an active AutoCirculate channel.
    - New `GetConnectedInput` function — to discover the `NTV2InputCrosspoint` a given `NTV2OutputCrosspoint` is feeding.
    - New `EnableHDMIHDRDolbyVision` and `GetHDMIHDRDolbyVisionEnabled` functions.
    - `GetFBSizeAndCountFromHW` is now private.
    - New **“Device Features” API** — **NOTE:** _work-in-progress_ — We would eventually like to eliminate the dichotomy between the what the SDK claims a device can do (determined at compile-time) and what the device can actually do (determined by its firmware at run-time).
  - New `j2kDecoderConfig` and `j2kDecoderConfig` classes
  - `CNTV2ConfigTs2022`, `CNTV2MailBox` and `CNTV2MBController` classes — many changes
  - `TSGenerator`, `PESGen`, `PATGen`, `PMTGen`, and `ADPGen` classes replaced the `CNTV2MBController` class
  - **Device Features API:**
    - `NTV2DeviceCanDoJ2K` function
    - `NTV2DeviceCanDo2110` function
  - **Driver Changes:**
    - All Platforms
      - Virtual register space has been expanded to store up to 1,024 32-bit values. Clients can use `WriteRegister`/`ReadRegister` to store/retrieve arbitrary 4-byte values in “registers” starting at `kVRegFirstOEM` up to `kVRegLast`.
      - Bug fixes for timecode, VITC and Anc ingest/playout.
    - Linux
      - Improved DMA transfer performance from multithreaded applications using AJA devices having NWL DMA engines.
      - The ancient “xena” name for the kernel module (`.ko`) has finally been replaced with “ajantv2” to match the naming used on macOS and Windows.

## SDK 13.1 changes (from 13.0)

- **KONA-IP:** SMPTE 2022 IGMP v3 Implementation — When the source IP address is included in the Rx match selection, source specific IGMP subscriptions are made.
- **API Changes:**
  - New `CNTV2Card` functions `GetRunningFirmwareDate`, `GetRunningFirmwareTime` and `GetRunningFirmwareRevision` functions — to detect if the installed device firmware is the firmware that‘s currently running.

## SDK 14.0 changes (from 13.1)

- Introducing **Io4K-Plus** — Thunderbolt III, 4-lane gen3 PCIe, 6G/12G SDI input/output, XLR microphone input, HDMI 2.0
- Introducing **IoIP** — Thunderbolt III, 4-lane gen3 PCIe, HDMI 2.0
- **Audio Mixer for Kona 4, Io4K and Io4K+ (Quad firmware only)**
  - Note that on Kona4 and Io4K, quadrant-mode HMDI monitoring for 4K/UHD was removed to fit the new audio mixer functionality into the FPGA.
- **API Changes:**
  - `NTV2FormatDescriptor`:
    - More planar support: new `PlaneToString` and `ByteOffsetToPlane` functions
    - Other changes:
      - New `ByteOffsetToRasterLine`, `RasterLineToByteOffset` and `IsAtLineStart` functions.
      - New `GetLineOffsetFromSMPTELine` function.
      - New `IsSDFormat` function.
  - `CNTV2Card`:
    - New `DeviceCanDoAudioMixer` and `DeviceHasMicInput` functions, to detect if the audio mixer firmware and/or microphone input hardware is present or not.
    - New `GetAudioMixer…` and `SetAudioMixer…` functions — to support the new audio mixer firmware in the Io4K and Io4K-Plus.
    - New `GetSDIInput6GPresent` and `GetSDIInput12GPresent` functions — to support the new 6G/12G SDI connectors on the Io4K-Plus.
    - New `GetSDIOut6GEnable`/`SetSDIOut6GEnable` and GetSDIOut12GEnable`/`SetSDIOut12GEnable` functions — to support the new 6G/12G SDI connectors on the Io4K-Plus.
    - New `GetAnalogAudioIOConfiguration`/`SetAnalogAudioIOConfiguration` functions — to support the new analog audio connector on the Io4K-Plus.
    - New `CanConnect` function now implemented - validates internal xpt routes.
    - New `GetEnableHDMIOutUserOverride`, `EnableHDMIOutUserOverride`, `GetEnableHDMIOutCenterCrop` and `EnableHDMIOutCenterCrop` functions (only for devices that support HDMI 2.0).
    - New `VirtualDataWrite` and `VirtualDataRead` functions writing/reading HDMI metadata.
    - New `GetAncExtractorRunState` and `GetAncInserterRunState` functions (for devices with custom Anc inserter/extractor firmware).
  - `AUTOCIRCULATE_TRANSFER`:
    - New `acHDR10PlusDynamicMetaData` member for optionally transmitting HDR10+ dynamic metadata per-frame.
  - **Device Features API**
    - New `NTV2DeviceCanDo12GOut` function
    - New `NTV2DeviceHasSPIv5` function
    - New `NTV2DeviceCanDoAudioMixer` function
    - New `NTV2DeviceHasLEDAudioMeters`, `NTV2DeviceHasHeadphoneJack`, `NTV2DeviceHasAudioMonitorRCAJacks`, and `NTV2DeviceHasBiDirectionalAnalogAudio` functions
    - New `NTV2DeviceGetAudioMixerSystem`, `NTV2DeviceCanDoAudioOut` and `NTV2DeviceCanDoAudioIn` functions
  - **Miscellaneous**
    - New `NTV2StandardSet` collection and `NTV2DeviceGetSupportedStandards` function.
    - New `FindFirstMatchingRegisterNumber` and `GetNormalizedFrameGeometry` functions.
    - New `GetFirstMatchingVideoFormat` and `GetNormalizedFrameGeometry` functions.
    - New `NTV2IpErrorEnumToString` function.
  - `CNTV2SignalRouter`
    - New `ResetFromRegisters`, `GetWidgetIDs`, `GetAllWidgetInputs` and `GetAllRoutingRegInfo` functions.
  - `CNTV2Config2022`, `CNTV2Config2110` and `CNTV2MBController` classes & cousins
    - _extensive changes — too many to list_
  - Enums
    - `NTV2Stream` has new `NTV2_METADATA_STREAM` and `NTV2_UNUSED_STREAM` values (`NTV2_AUDIO2_STREAM` was dropped).
    - `NTV2AudioSource` has a new `NTV2_AUDIO_MIC` value.
    - `NTV2InputAudioSelect` has a new `NTV2_MicInSelect` value.
    - `NTV2SDITransportType` has new `NTV2_SDITransport_6G` and `NTV2_SDITransport_12G` values.
    - `NTV24kTransportType` has a new `NTV2_4kTransport_12g_6g_1wire` value.
    - New `NTV2WidgetID` values: `NTV2_Wgt12GSDIIn1…4` and `NTV2_Wgt12GSDIOut1…4`, `NTV2_WgtHDMIIn1v4` and `NTV2_WgtHDMIOut1v4`.
    - New `NTV2AudioMixerChannel` enum.
    - New `NTV2AnalogAudioIO` enum to describe Io4K+’s analog audio connector configuration (8-in, 8-out, 4-in/4-out, or 4-out/4-in).
    - The `VPIDStandard` values have changed.
    - New `NTV2IpError` enum to describe IP errors.
    - Changed several `NTV2FrameBufferFormat`s that were unused in our supported devices to support new 3-plane planar formats in the Corvid44:
      - `NTV2_FBF_8BIT_QREZ` is now `NTV2_FBF_8BIT_YCBCR_420PL3`;
      - `NTV2_FBF_UNUSED_23` is now `NTV2_FBF_8BIT_YCBCR_422PL3`;
      - `NTV2_FBF_UNUSED_26` is now `NTV2_FBF_10BIT_YCBCR_420PL3_LE`;
      - `NTV2_FBF_UNUSED_27` is now `NTV2_FBF_10BIT_YCBCR_422PL3_LE`;
      - Renamed the 2-plane planar formats — supported only by the Corvid HEVC — to clarify they have only two planes:
        - `NTV2_FBF_10BIT_YCBCR_420PL` is now `NTV2_FBF_10BIT_YCBCR_420PL2`;
        - `NTV2_FBF_10BIT_YCBCR_422PL` is now `NTV2_FBF_10BIT_YCBCR_422PL2`;
        - `NTV2_FBF_8BIT_YCBCR_420PL` is now `NTV2_FBF_8BIT_YCBCR_420PL2`;
        - `NTV2_FBF_8BIT_YCBCR_422PL` is now `NTV2_FBF_8BIT_YCBCR_422PL2`.
- **Driver Changes:**
  - All Platforms: New `NTV2VirtualData` message.
  - macOS: HDMI input detection moved from the ‘**AJAAgent**’ service daemon into the driver. This comports with Linux and Windows.
- **Kona IP Changes**
  - Support is added for SMPTE 2110.  Beta – video Tx & audio Tx only (no Anc, no Rx)  
    - A new API class `CNTV2Config2110` is used to set up the card.  The 2110 firmware is designed to operate locked to PTP timing.
    - On an experimental basis, support is added for use of SDPs with 2110. For Tx, SDPs are served using HTTP protocol for active channels. For Rx, SDPs can be obtained using HTTP directly by the KonaIP card or from a file given to the Config class. Thes SDPs are parsed and used to configure Rx channels for video and audio.
    - _This is an emerging technology and the current implementation should be regarded as experimental._
  - All KonaIP firmware versions now support IGMP v3.  For Rx when a 'source IP match' is enabled, a V3 IGMP source subscription will be made for the source IP address.
  - The 2022 firmware package supports both 2022-6 and 2022-7 operation.
    - 2022-7 is now configured globally for all channels.  Also, the network path differential is global rather than per channel as it was previously
    - For 2022-6 operation, inputs and outputs can be assigned to either link-A (top SFP) or link-B (bottom SFP).  Previously the assignments were fixed and different firmware was required to use link-B for 2022-6.  In this release all channels can be routed to either SFP link. 
  - The `ntv2konaipjsonsetup` program has been enhanced for both 2022 and 2110 use.  For 2022 J2K, it should be used in conjunction with the `ntv2konaipj2ksetup` program.  A `-k` option has been added which lists the fields supported for each protocol.  The changes in 2022-7 and 2022-6 configuration described above are reflected in these programs too.
  - For both 2022 and 2110 Config classes, when the IP addresses are configured, the Config class validates that the installed firmware is a match for the SDK. If there is a mismatch then the IP addresses are not configured and the call returns false.  This ensures that there is a fimware/software match and effectively disables the IP I/O for the board on a mismatch.
  - Functions have been added to 2022 and 2110 Config classes to gather information on SFPs.

## SDK 14.2 & 14.3 changes (from 14.0)

- **New Device Support:**
  - **KONA 1** — A modern replacement for the Corvid 1
  - **KONA HDMI** — 4-input HDMI capture card
- **API Changes:**
  - `CNTV2Card`:
    - New `DMAReadAnc`/`DMAWriteAnc` methods;
    - `GetHDMIInputVideoFormat`, `GetHDMIInputStatusRegister` and `GetHDMIInputColor` now accept an optional `NTV2Channel` number.
  - `NTV2_POINTER`:
    - New methods to **Set**/**Get** to/from `std::vector`s of `uint64_t`, `uint32_t`, `uint16_t` and `uint8_t`.
    - New `Dump` method
  - New `CNTV2SupportLogger` class — Generates NTV2 support log files for debug purposes
  - `AJAAncillaryList` and `AJAAncillaryData`: — many changes to support SMPTE 2110 Ancillary data streams.
  - Tsi (2-sample interleave) is now the default scheme for 4K/UHD over 3G SDI on boards newer than Kona3G/Corvid24 (Kona4, Io4K, Corvid44, etc.).
  - Removed `using namespace` statement in `ntv2rp188.h`
- **Demo Apps**
  - **NTV2Capture** — Tweaked to work with the new Kona HDMI.
  - **NTV2Capture4K** — Now can capture Tsi instead of just quads/squares.
  - **NTV2CCPlayer** — Fixed _schmutz_ in top lines of video when using `--vanc` command-line option.
  - **NTV2KonaIPJSONSetup** — Many changes to support new IP capabilities.
  - **NTV2Player4K** — Now can play 4K using Tsi instead of only quads/squares. Now works with KonaIP.
  - **NTV2QtCapture** & **NTV2QtMultiInput** — Improvements and bug fixes in `AJAPreviewWidget` and `NTV2FrameGrabber` classes.

## SDK 15.0 changes (from 14.3)

- The ‘**ajabase**’ and ‘**ajaanc**’ class libraries have been merged into the ‘**ajantv2**’ library.
  - On Windows, this may require linking with additional Windows libraries.
  - On Linux, this may require linking with additional libraries (e.g. `-lrt`).
- **KONA 5** — our first Gen3×8 PCIe ingest/playout board with 4GB SDRAM and 4×12Gbps SDI connectors
- **Two-Sample-Interleave (Tsi) 4K video formats for the 12g KONA 5** — which greatly simplifies 4K routing:
  - Just call `SetVideoFormat` with one of the new Tsi video formats (e.g. `NTV2_FORMAT_4096x2160p_2400`).
  - “Square Division” mode is now the exception. Call `SetVideoFormat` with one of the “4x” video formats (e.g. `NTV2_FORMAT_4x2048x1080p_2400`).
- Most `CNTV2Card` functions that return a value using a pointer parameter have been deprecated. We’ve standardized on the use of non-constant reference parameters instead.

## SDK 15.2 changes (from 15.1)

- **New Device Support:**
  - **Corvid44-12G** — OEM version of **KONA 5** — 4×12g connectors (but no HDMI out or breakout cable/box)
- **UHD2 Support** — UHD2 (4×UHD) ingest/playout support for **KONA 12G** and **Corvid44-12G**
- **S-2110 Anc Support:** — Seamless ancillary data stream support for **KONA IP** and **IoIP**.
- **API Changes:**
  - The `NTV2DeviceID` of the **KONA 5** (12G firmware) has changed from `DEVICE_ID_KONA5_12G` to `DEVICE_ID_KONA5_4X12G`.
  - The `NTV2DeviceID` of the **Corvid44-12G**, currently `DEVICE_ID_CORVID44_12G`, is subject to change.
  - **Demos:**/
    - **NTV2Capture4K** now emits ancillary data by default.
    - New **NTV2Capture8K** and **NTV2Player8K** demos.
  - **‘ajaanc’ Library:**
    - Supporting S2110 Ancillary Data streams for **KONA IP** and **IoIP** has necessitated some API changes to this library.
    - `AJAAncillaryList`:
      - `SetFromIPAncData` and `SetFromSDIAncData` have been deprecated in favor of the new, generic `SetFromDeviceAncBuffers` function.
      - `GetSDITransmitData` has been deprecated in favor of the SDI-default `GetTransmitData` function. If RTP is needed/desired, then the `GetIPTransmitData` function can be used instead.
      - The “Analog” (“raw”) ancillary data type map is now global (instead of per-`AJAAncillaryList` instance).
      - New thread-safe global setting API (`IsIncludingZeroLengthPackets`, `SetIncludeZeroLengthPackets`, `GetExcludedZeroLengthPacketCount`, `ResetExcludedZeroLengthPacketCount`) that controls whether or not `AddVANCData`/`AddReceivedAncillaryData` will add zero-length packets.
    - `AJAAncillaryDataLocation`:
      - Use of `AJAAncillaryDataSpace` has been deprecated in favor of the new, special Horizontal Offset and Line Number constants (e.g. `AJAAncDataHorizOffset_AnyHanc`).
      - `Set` function (with 6 parameters, one of which is ignored now) has been deprecated in favor of using the existing `SetDataLink`, `SetDataStream`, `SetDataChannel`, etc. functions.
    - `AJAAncillaryData`:
      - Multi-parameter versions of `SetDataLocation`, `GetDataLocation` (and friends) have been deprecated in favor of independently manipulating/interrogating `AJAAncillaryDataLocation`.
      - Added 32-bit frame identifier and `AJAAncillaryBufferFormat` members to `AJAAncillaryData` instances.
  - `CNTV2Card`:
    - `SetRP188Source` has been renamed `SetRP188SourceFilter` to better reflect what it actually does.
    - Added new `GetRP188BypassSource`/`SetRP188BypassSource` functions.
    - Added new `AUTOCIRCULATE_WITH_HDMIAUX` option to enable ingest/playout of **HDMI InfoFrame auxiliary data** (e.g. HDR data).
    - Added new `GetQuadQuadSquaresEnable`/`SetQuadQuadSquaresEnable` functions to support UHD2.
    - `DMAReadAnc`/`DMAWriteAnc` now have an optional `NTV2Channel` parameter for S2110 IP devices to specify which Anc FrameStore and extractor/inserter is being used.
    - Added new `AncInsertSetIPParams` function for configuring the S2110 ancillary data inserter.
    - Added new `GetMixerFGMatteEnabled`/`SetMixerFGMatteEnabled`, `GetMixerBGMatteEnabled`/`SetMixerBGMatteEnabled`, and `GetMixerMatteColor`/`SetMixerMatteColor` functions.
    - Added new `GetAudioOutputAESSyncModeBit`/`SetAudioOutputAESSyncModeBit` functions for falsifying the AES “Sync Mode” in outgoing embedded SDI audio.
    - Added new `DMABufferLock`/`DMABufferUnlockAll` functions for long-term locking of host buffer memory in kernel space.
    - Added new `SetVPIDTransferCharacteristics`, `SetVPIDColorimetry` and `SetVPIDVPIDLuminance` functions.
  - **Device Features:**
    - Added new `NTV2DeviceCanDoWarmBootFPGA` and `NTV2DeviceCanDo8KVideo` functions.
  - **Enum Types:**
    - Added new `NTV2VPIDTransferCharacteristics`, `NTV2VPIDColorimetry` and `NTV2VPIDLuminance` enum types.
  - `NTV2FormatDescriptor`:
    - Added new `GetWriteableRowAddress` function.
  - `NTV2_POINTER`:
    - New `Dump` function (to `std::string`).
  - `AUTOCIRCULATE_TRANSFER`:
    - `SetAllOutputTimeCodes` can now be told whether to set VITC2 or not.
  - **Utilities:**
    - Added new `NTV2DeviceGetSupportedGeometries`, `HasVANCGeometries`, `GetRelatedGeometries`, `GetVANCModeForGeometry`, `GetStandardFromGeometry` functions.
    - Added new `Is8KFormat` function.
    - Added new `NTV2MixerKeyerModeToString`, `NTV2MixerInputControlToString`, and `NTV2VideoLimitingToString` functions.
    - `GetFirstMatchingVideoFormat` now takes into account psf.
  - `NTV2TestPatternGen`:
    - Added new 12-bit RGB test patterns (**Zone Plate**, **Linear Ramp**, **HLG Narrow**, **PQ Narrow**, and **PQ Wide**).
  - **Transcode:** — New functions:
    - `ConvertLine_8bitABGR_to_48bitRGB`
    - `ConvertLine_8bitABGR_to_10bitABGR`
    - `ConvertLine_2vuy_to_yuy2`
    - `ConvertLine_8bitABGR_to_10bitRGBDPXLE`
    - `ConvertLine_8bitABGR_to_24bitRGB`
    - `ConvertLine_8bitABGR_to_24bitBGR`

## SDK 15.2.2 changes (from 15.2)

- **API Changes:**
  - `CNTV2Card::SetFrameBufferFormat` has three new, optional parameters for setting the HDR characteristics of the FrameStore/Channel, which allows the driver to properly decorate the output VPID for any output fed by the FrameStore. They default to `NTV2_VPID_TC_SDR_TV`, `NTV2_VPID_Color_Rec709` and `NTV2_VPID_Luminance_YCbCr`.
  - `CNTV2Card::GetNTV2VideoFormat` was changed to correctly detect 1080i/psf 23.98/24.00 fps dual-link 3G RGB. Unfortunately, this breaks 1080i@4798/4800 detection, since they’re electrically identical. This will be corrected in a future SDK by consulting the input VPID.

## SDK 15.5 changes (from 15.2.2)

- **New Firmware Bitfiles:**
  - **Corvid44-12G 8KMK Bitfile** — `DEVICE_ID_CORVID44_8KMK` replaces `NTV2DeviceID DEVICE_ID_CORVID44_12G` — same capabilities:
    - Up to 8K YUV in/out
    - Up to 4K using CSCs & mixer/keyers
  - **Corvid44-12G 8K Bitfile** — `DEVICE_ID_CORVID44_8K`:
    - Up to 8K YUV or RGB in/out
    - No CSCs, no Mixer/Keyers
  - **KONA 5 8KMK Bitfile** — `DEVICE_ID_KONA5_8KMK`:
    - Up to 8K YUV in/out
    - Up to 4K using CSCs & mixer/keyers
    - No audio mixer
  - **KONA 5 8K Bitfile** — `DEVICE_ID_KONA5_8K` replaces `NTV2DeviceID DEVICE_ID_KONA5_4X12G` — same capabilities:
    - Up to 8K YUV or RGB in/out
    - No CSCs, no Mixer/Keyers, no audio mixer
- **API Changes:**
  - `AJAAncillaryList`:
    - Added new overloaded function `AddAncillaryData` for appending packets from another `AJAAncillaryList`.
    - Added new `GetIPTransmitDataLength` function.
  - `AJAAncillaryData`:
    - Added new `IS_KNOWN_AJAAncillaryDataType` macro.
    - Numerous changes to the `AJARTPAncPayloadHeader` class.
  - `CNTV2Card`:
    - New `NTV2_POINTER`-based version of the `DMABufferLock` function.
    - New `DMAClearAncRegion`, `GetAncRegionOffsetAndSize` and `GetAncRegionOffsetFromBottom` functions.
    - Added missing `GetRawAudioTimer` function.
    - All-new, corrected **Audio Mixer API**, that also introduces new functions to get/set muting for the mixer’s output audio channels. The original input-specific APIs introduced in SDK 15.2 will be deprecated.
    - `AutoCirculateInitForInput`, `AutoCirculateInitForOutput` and `FindUnallocatedFrames` now use `UWord`s instead of `UByte`s for frame numbers, in order to handle frame numbers over 255.
    - New `GetConnectedInputs` routing function.
    - `GetLTCInputPresent` can now answer for a second LTC input port, for devices that have them.
    - `AncExtractGetDefaultDIDs` can now optionally return DIDs for SD audio packets instead of just HD ones.
    - It was necessary to remove the `LocalLoadBarsTestPattern` function.
  - **Device Features:**
    - Added new `NTV2DeviceCanDoHFRRGB` function to test if the device supports RGB 1080p at more than 50Hz.
  - **Data Types:**
    - `DEVICE_ID_CORVID44_12G` has been deprecated and replaced by `DEVICE_ID_CORVID44_8KMK`.
    - Added new `DEVICE_ID_CORVID44_8K` device ID for the new bitfile that supports up to 8K YUV or RGB in/out with no CSCs or Mixer/Keyers.
    - `DEVICE_ID_KONA5_4X12G` has been deprecated and replaced by `DEVICE_ID_KONA5_8K`.
    - Added new `DEVICE_ID_KONA5_8KMK` device ID for the new **KONA 5** bitfile.
    - Added new `NTV2_FBF_12BIT_RGB_PACKED` frame buffer format, an AJA-specific 12-bit uncompressed RGB format.
    - Added new `NTV2_IS_UHD2_FULL_VIDEO_FORMAT` macro.
    - Enumerated types and constants that were tied to AJA’s “retail” software — not the OEM SDK — have been consolidated and slated for deprecation under the `R2_DEPRECATE` macro.
    - Added new `NTV2AncillaryDataRegion` enum.
    - Added new `UByteSequence` data type, a `std::vector` of `UByte` values.
    - Added new `HDRDriverValues` struct.
    - Defined new `RP188SourceFilterSelect` to eventually replace the poorly-named `RP188SourceSelect` enum type.
  - `NTV2FormatDescriptor`:
    - SMPTE line numbering fixes for 4K & 8K standards.
    - Added support for new `NTV2_FBF_12BIT_RGB_PACKED` pixel format.
  - `AUTOCIRCULATE_STATUS`:
    - Added handy new `GetMode` function.
  - **Utilities:**
    - Added new `NTV2BreakoutTypeToString` and `NTV2AncDataRgnToStr` functions.
    - `GetNTV2HDMIInputSourceForIndex` is now slated for eventual deprecation.
  - `NTV2TestPatternGen`:
    - Added new `DrawTestPattern` function that conveniently accepts an `NTV2FormatDescriptor`.
    - Added handy new `canDrawTestPattern` function.
    - Added new `NTV2_IS_VALID_PATTERN` macro.
  - **Transcode:**
    - Added new `Convert16BitARGBTo12BitRGBPacked` function.
  - `CNTV2VPID`:
    - Added new “to string” conversion functions: VersionString, StandardString, PictureRateString, SamplingString, ChannelString, DynamicRangeString, BitDepthString, LinkString, AudioString.
- **Demo Changes:**/
  - Removed redundant switching of bidirectional SDI connector to receive mode in `NTV2Capture::SetupVideo`.
  - **NTV2CCPlayer** can optionally force multiple RTP packets per-frame on IP devices.
  - **NTV2Player4K** has been cleaned up and now shows how to play 4K through 6G and 12G SDI spigots on **Io4K+**, **KONA 5** and **Corvid44-12g**.
  - Fixed several compile-time warnings.
- **Driver Changes:**
  - HDMI input detection improvements.
  - Added HDR info to SDI output VPID.
  - Added ability to configure maximum PCIe read request size to improve 8K transfer speed for **KONA 5**, **Corvid44-12G**.
  - Linux: Fixed a serious long-standing race condition in the Linux driver that could cause multiple processes waiting on the same interrupt (e.g. `eOutput1`) to time-out.
  - Windows: Fixed a serious issue in the audio driver that could cause blue screens when system audio output is routed through the AJA device.

## SDK 16.0 changes (from 15.5)

**New Device Support:**
  - **T-Tap Pro** — A very much improved 4K-capable version of the T-Tap
- **API Changes:**
  - `CNTV2Card`:
    - To improve 8K/UHD2 DMA performance: There’s a new “map” option on the `DMABufferLock` functions to have the driver also lock down the segment map in kernel memory;  New `DMABufferUnlock` function to unlock buffers locked by `DMABufferLock`;  New `DMABufferAutoLock` function.
    - To better accommodate new 8K/UHD2 and 12-bit workflows, we’ve introduced **Fast Bitfile Switching** (aka **“dynamic reconfig”**) and new per-device bitfile sets for **KONA 5** and **Corvid44-12G**. This includes a new **switchbitfile** command-line tool, and new “dynamic device" API functions (e.g. `IsDynamicDevice`, `LoadDynamicDevice`, etc.).
    - New `GetDeviceFrameInfo` and `DeviceAddressToFrameNumber` functions.
    - New `GetTsiMuxSyncFail` function (requires firmware support).
    - The _“inFrameCount”_ parameter of `AutoCirculateInitForInput` and `AutoCirculateInitForOutput` functions is now ignored if a legitimate frame range is specified in the _“inStartFrameNumber”_ and _“inEndFrameNumber”_ parameters.
    - New `ApplySignalRoute` function that accepts a `std::map` of input-to-output crosspoint connections.
    - New **SDI Bypass Relay API** that deprecates the old per-relay functions, replacing them with generic functions that have an index parameter for specifying the relay (connector-pair).
    - New `GetBitfileInfoString` function.
    - New HDR “getter” functions:  `GetVPIDTransferCharacteristics`, `GetVPIDColorimetry`, `GetVPIDVPIDLuminance`.
    - Several methods now accept `NTV2ChannelSet`s, for configuring multiple channels or SDI connectors in one API call.
  - **Signal Router:**
    - Added new `ResetFrom` function to reset the `CNTV2SignalRouter` instance from a given `NTV2XptConnections` map.
    - Added new `GetConnectionsFromRegs`, `CompareConnections` and `CreateFromString` class methods.
  - **Data Types:**
    - Defined new `NTV2XptConnection` to augment the `NTV2Connection` pair.
    - Defined new `NTV2XptConnections` to replace the poorly-named `NTV2ActualConnections` mapping.
  - `NTV2FormatDescriptor`:
    - New `GetTotalBytes` function returns the total number of bytes required to hold the raster, including all planes of planar formats (which used to require multiple calls to `GetTotalRasterBytes`).
  - `AUTOCIRCULATE_STATUS`:
    - Added handy new `OptionFlags`, `WithCustomAnc`, `WithRP188`, `WithLTC`, `IsFieldMode` and `WithHDMIAuxData` functions.
  - **Utilities:**
    - Added new `GetNTV2FrameRateFromNumeratorDenominator` function.
    - Enlarged _rowBytes_ parameters of `Make10BitBlackLine`, `Make10BitWhiteLine`, `Make10BitLine`, `SetRasterLinesBlack`, and `CopyRaster` functions to `ULWord`s (`uint32_t`) to accommodate larger 8K/UHD2 rasters.
  - **Misc:**
    - **C++11:** — This is AJAʼs first SDK that requires a C++11 compliant compiler by default. Instructions on how to build the SDK without C++11 compiler support are in the **Getting Started** guide.
    - Added `SetLLDPInfo` and `GetLLDPInfo` functions to `CNTV2Config2110` class.
    - Added **MultiLinkOut** virtual widget in 12G-routable devices (**KONA 5**, **Corvid44-12G**).
  - **Transcode:**
    - Added new `Convert16BitARGBTo12BitRGBPacked` function.
  - `CNTV2VPID`:
    - Added new “to string” conversion functions: `VersionString`, `StandardString`, `PictureRateString`, `SamplingString`, `ChannelString`, `DynamicRangeString`, `BitDepthString`, `LinkString`, `AudioString`.
  - **Demos:**
    - All:
      - The `-b|--board` command line option used in most demos has been deprecated in favor of `-d|--device`.
      - Specifying `?` for  `-d|--device` lists all available devices and exits the demo.
      - Enhanced logging: All capture demos log into a new **DemoCapture** category; playout demos log into a new **DemoPlayout** category.
      - **NTV2Burn** and **NTV2LLBurn** now use new **Bypass Relay API** (see above).
      - **NTV2LLBurn** now demonstrates a new custom HANC playout capability available on some OEM devices.
    - **NTV2Capture:**
      - Now grabs all available input timecodes. When anc capture is enabled, fixed `--anc` to record the number of F1 & F2 anc bytes captured per-frame.
      - The `-a` option still enables anc capture, but the `--anc` option now requires a _filePath_ parameter, and it records the raw captured anc data into a binary data file (that can be played using **ntv2player**).
    - **NTV2Player:**
      - Added a new `--anc` option that will inject/play raw anc data supplied by the binary data file specified as its parameter. (**ntv2capture** can now create such binary anc files.)
    - **NTV2OutputTestPattern:**
      - `-p|--pattern` option now accepts pattern names or standard web color names.
      - New `-v|--videoFormat` option for specifying video format to override current device video format.
      - New `--vanc` option for specifying vanc mode.
      - New `--pixelFormat` option for specifying frame buffer pixel format.
  - **Driver:**
    - Numerous DMA performance improvements to support 8K/UHD2 on **KONA 5**, **Corvid44-12G**.

## SDK 16.2 changes (from 16.0)

- **New Device Support:**
  - **IoX3** — `DEVICE_ID_IOX3 — A modern replacement for the **IoXT**
- **Improved 12-Bit RGB and 4:4:4 workflow support for 4K/UHD (Kona5, Corvid44-12G)**
- **Expanded 64/32 audio channel support over multi-link SDI (Kona5, Corvid44/12G, Corvid88 and Corvid44)**
- **API Changes:**
  - Bumped the actively-deprecated APIs and data types from SDK 14.3 to 15.2. If your code compiled under SDK 16.1, and fails to compile under SDK 16.2:
    1. Scroll up to “SDK 15.0”, then apply each listed change to your code, if applicable.
    1. Repeat step 1 for “SDK 15.1” and “SDK 15.2”.
  - `CNTV2Card`:
    - New detailed `GetDeviceFrameInfo` overload.
    - New `AncInsertGetReadInfo` and `AncExtractGetWriteInfo` member functions.
    - New overloads for `SetVANCShiftMode`, `SetNumberAudioChannels`, `SetAudioBufferSize`, `SetAudioLoopBack`, and `SetSDIOutputAudioSystem` to operate on more than one channel, audio system or widget in a single function call.
    - Deprecated `GetActiveFrameDimensions`, `GetNumberActiveLines`, `SetPCIAccessFrame`, `GetPCIAccessFrame` and `FlipFlopPage`.
    - Members `GetBaseAddress`, `GetRegisterBaseAddress` and `GetXena2FlashBaseAddress` first deprecated in SDK 16.0 now produce compile-time warnings when used.
  - New `SDRAMAuditor` class used by `CNTV2SupportLogger` to detect device SDRAM utilization clashes or conflicts.
  - New `NTV2BitfileHeaderParser` class is now used commonly by all firmware-related classes for bitfile header parsing.
  - `NTV2TestPatternGen`:
    - Two overloaded members `DrawTestPattern`, first deprecated in SDK 16.0, now produce compile-time warnings when used.
  - `NTV2FormatDescriptor`:
    - New `GetVideoWriteSize` function.
  - **Utilities:**
    - New `GetAudioSamplesPerSecond`, `NTV2BitfileTypeToString` functions.
  - **Device Features API:**
    - New `NTV2DeviceCanDoMultiLinkAudio`, `NTV2DeviceGetNumLUTBanks` functions.

## SDK changes after 16.2

The NTV2 SDK became open-source as of version 17.0.
See the open-source documentation for SDK changes after 17.0.
