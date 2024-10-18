
# Porting From “NTV2RoutingEntry” to “NTV2InputCrosspointID”

This article details how to port an application that uses the old `NTV2RoutingEntry` structs to use the new `NTV2InputCrosspointID`’s that were introduced in SDK version 12.3.

> **NOTE:** `NTV2RoutingEntry` was first deprecated in **SDK 12.5**. Code that uses `NTV2RoutingEntry`s in calls to `CNTV2Card::Connect`, or `CNTV2SignalRouter::AddConnection` must be ported as described herein.

In most cases, you can easily port your code by …
1. Globally search for function calls that start with “Get” (or “::Get”) and that also end with “SelectEntry()”.
1. Replace “Get” (or “::Get”) with “NTV2_Xpt”, and delete “SelectEntry()”.

## ‘CNTV2SignalRouter’ Example

### Old ‘NTV2RoutingEntry’-Based API:
```
. . .
CNTV2Card         device (0);  // The device
CNTV2SignalRouter router;      // My signal router
. . .
router.AddConnection (::GetDualLinkIn1InputSelectEntry(),   NTV2_XptSDIIn1);
router.AddConnection (::GetDualLinkIn1DSInputSelectEntry(), NTV2_XptSDIIn1DS2);
router.AddConnection (::GetFrameBuffer1InputSelectEntry(),  NTV2_XptDuallinkIn1);
router.AddConnection (::GetFrameBuffer2InputSelectEntry(),  NTV2_XptDuallinkIn1);
device.ApplySignalRoute (router);
. . .
```
### New ‘NTV2InputCrosspointID’-Based API:
```
. . .
CNTV2Card         device (0); // The device
CNTV2SignalRouter router;     // My signal router
. . .
router.AddConnection (NTV2_XptDualLinkIn1Input,   NTV2_XptSDIIn1);
router.AddConnection (NTV2_XptDualLinkIn1DSInput, NTV2_XptSDIIn1DS2);
router.AddConnection (NTV2_XptFrameBuffer1Input,  NTV2_XptDuallinkIn1);
router.AddConnection (NTV2_XptFrameBuffer2Input,  NTV2_XptDuallinkIn1);
device.ApplySignalRoute (router);
. . .
```

## ‘CNTV2Card’ Example

### Old ‘NTV2RoutingEntry’-Based API:
```
. . .
CNTV2Card device(0);  // The device
. . .
device.Connect (::GetDualLinkIn1InputSelectEntry(),   NTV2_XptSDIIn1);
device.Connect (::GetDualLinkIn1DSInputSelectEntry(), NTV2_XptSDIIn1DS2);
device.Connect (::GetFrameBuffer1InputSelectEntry(),  NTV2_XptDuallinkIn1);
device.Connect (::GetFrameBuffer2InputSelectEntry(),  NTV2_XptDuallinkIn1);
. . .
```
### New ‘NTV2InputCrosspointID’-Based API:
```
. . .
CNTV2Card device(0);  // The device
. . .
device.Connect (NTV2_XptDualLinkIn1Input,   NTV2_XptSDIIn1);
device.Connect (NTV2_XptDualLinkIn1DSInput, NTV2_XptSDIIn1DS2);
device.Connect (NTV2_XptFrameBuffer1Input,  NTV2_XptDuallinkIn1);
device.Connect (NTV2_XptFrameBuffer2Input,  NTV2_XptDuallinkIn1);
. . .
```
