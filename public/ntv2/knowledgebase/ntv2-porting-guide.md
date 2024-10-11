# NTV2 Porting Guide

This article suggests how to port application code that builds against an older NTV2 SDK to the most recent SDK.

_(Last updated:  October, 2024  --  to include SDK 17.1)_

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

### API changes past SDK 15.0 are documented in the “Getting Started” online documentation for the SDK
1. Go to The **NTV2_DEPRECATE Macros** section.
1. Scroll down through the deprecation notices.
