# Porting “CNTV2Task” to New “AutoCirculateTransfer” API

This article describes how to port an application that uses the old `CNTV2Task`/`AutoCircTimeCodeTask` API to frame-accurately capture/playout timecode to the newer `AUTOCIRCULATE_TRANSFER`/`AutoCirculateTransfer` API that was introduced in SDK version 12.3.

_(This article applies to SDK versions before 12.3)_

> **NOTE:** You only need to do this if your application:\
• Uses AutoCirculate to capture or play video\
• Must run on version 12.4 or later NTV2 drivers\
• Requires frame-accurate timecode to/from SDI spigots that don’t correspond to the AutoCirculate channel that’s being used (e.g., you’re AutoCirculating Channel 2, but need to embed/deembed timecode to/from SDI 1).

If your AutoCirculate-based application only sets/gets timecode to/from the SDI spigot that corresponds to the AutoCirculate channel, your application will continue to run with the NTV2 driver version 12.4.

> **Background:**  `CNTV2Task`/`AUTOCIRCULATE_TASK_STRUCT` was hacked into the SDK long ago to allow AutoCirculate clients to frame-accurately set/get timecode to/from SDI spigots other than the channel being circulated. In order to add support for embedded Field2 VITC for interlaced video in the 12.4 development, the Tasks hack wouldn’t scale … which added a lot of unnecessary complexity to the driver. By dropping `CNTV2Task`/`AUTOCIRCULATE_TASK_STRUCT`, the driver is now simpler, more reliable and easier to test.

> **IMPORTANT:**  The new API’s structs use constructors to properly initialize themselves. **DO NOT use ‘memset’ or ‘bzero’ to clear them**, as this will undo much of what their constructors accomplish, and in most cases will leak memory. Please follow the coding patterns shown here.

## Capture Example

An application captures video using AutoCirculate channel 2, but FrameStore 2 is routed to receive video (with timecode) from SDI Input 1.

### Old CNTV2Task-Based API:
```
. . .
CNTV2Card                            device (0);
AUTOCIRCULATE_TRANSFER_STRUCT        inputXfer;       // My A/C input transfer info
AUTOCIRCULATE_TRANSFER_STATUS_STRUCT inputXferStatus; // My A/C input status
CNTV2Task                            inputTaskList;   // My task list for frame-accurate input timecode
::memset (&inputXfer, 0, sizeof (inputXfer));
::memset (&inputXferStatus, 0, sizeof (inputXferStatus));
inputXfer.channelSpec            = NTV2CROSSPOINT_INPUT2;
inputXfer.videoBuffer            = new UByte [myVideoBufferSize];
inputXfer.videoBufferSize        = myVideoBufferSize;
inputXfer.frameRepeatCount       = 1;
inputXfer.desiredFrame           = -1;
inputXfer.bDisableExtraAudioInfo = true;

device.StopAutoCirculate (inputXfer.channelSpec);
device.InitAutoCirculate (inputXfer.channelSpec, /*startFrame*/ 0, /*endFrame*/ 6, /* # chls */ 1, NTV2_AUDIOSYSTEM_1);

inputTaskList.Clear ();
inputTaskList.AddTimeCodeReadTask ();
device.SetAutoCirculateCaptureTask (inputXfer.channelSpec, inputTaskList);
device.StartAutoCirculate (inputXfer.channelSpec);

while (!mGlobalQuit)
{
   AUTOCIRCULATE_STATUS_STRUCT acStatus;
   device.GetAutoCirculate (inputXfer.channelSpec, &acStatus);
   if (acStatus.state == NTV2_AUTOCIRCULATE_RUNNING && acStatus.bufferLevel > 1)
   {
      //   Transfer from the device into our host AVDataBuffer...
      device.TransferWithAutoCirculate (&inputXfer, &inputXferStatus);

      // Clear the input task list
      inputTaskList.Clear ();

      //   Get the capture tasks for the transfered frame...
      FRAME_STAMP_STRUCT  acFrameStamp;
      device.GetFrameStampEx2 (inputXfer.channelSpec, inputXferStatus.transferFrame, &acFrameStamp, inputTaskList);

      //   Get the first task and confirm it is a time code read task...
      AutoCircGenericTask * pTask (inputTaskList.GetTask (0));
      if (pTask == NULL || pTask->taskType != eAutoCircTaskTimeCodeRead)
         continue;   // Failed -- try again later

      // Find the timecodes of interest inside pTask->u.timeCodeTask...
      // pTask->u.timeCodeTask.TCInOut1
      // pTask->u.timeCodeTask.LTCEmbedded
   }   //   if A/C running and frame(s) are available for transfer
   else
      device.WaitForInputVerticalInterrupt (NTV2_CHANNEL2);
} // loop til quit signaled

device.StopAutoCirculate (inputXfer.channelSpec);
. . .
```
### New API:
```
. . .
CNTV2Card               device (0);
AUTOCIRCULATE_TRANSFER  inputXfer;   // My A/C input transfer info 

device.AutoCirculateStop (NTV2_CHANNEL2);
device.AutoCirculateInitForInput (NTV2_CHANNEL2, /*#frames*/ 7, NTV2_AUDIOSYSTEM_INVALID, AUTOCIRCULATE_WITH_RP188);
device.AutoCirculateStart (NTV2_CHANNEL2);

while (!mGlobalQuit)
{
   AUTOCIRCULATE_STATUS acStatus;

   device.AutoCirculateGetStatus (NTV2_CHANNEL2, acStatus);
   if (acStatus.IsRunning () && acStatus.HasAvailableInputFrame ())
   {
      //  Transfer from the device into our host AVDataBuffer...
      device.AutoCirculateTransfer (NTV2_CHANNEL2, inputXfer);

      NTV2_RP188 tcSDI1, ltcSDI1;

      inputXfer.GetInputTimeCode (tcSDI1, NTV2_TCINDEX_SDI1);
      inputXfer.GetInputTimeCode (ltcSDI, NTV2_TCINDEX_SDI1_LTC);
   } // if A/C running and frame is available for transfer
   else
      device.WaitForInputVerticalInterrupt (NTV2_CHANNEL2);
} // loop til quit signaled

device.AutoCirculateStop (NTV2_CHANNEL2);
. . .
```
## Playout Example

An application plays video using AutoCirculate channel 3, but in addition to routing video (with timecode) to SDI Out 3, it also routes video (with same timecode) to SDI Out 4 and SDI Out 5.

### Old CNTV2Task-Based API:
```
. . .
CNTV2Card                            device (0);
AUTOCIRCULATE_TRANSFER_STRUCT        outputXfer;       // My A/C output transfer info
AUTOCIRCULATE_TRANSFER_STATUS_STRUCT outputXferStatus; // My A/C output status
CNTV2Task                            outputTaskList;   // My task list for frame-accurate output timecode
::memset (&outputXfer, 0, sizeof (outputXfer));
::memset (&outputXferStatus, 0, sizeof (outputXferStatus));
outputXfer.channelSpec            = NTV2CROSSPOINT_CHANNEL3;
outputXfer.videoBuffer            = new UByte [myVideoBufferSize];
outputXfer.videoBufferSize        = myVideoBufferSize;
outputXfer.frameRepeatCount       = 1;
outputXfer.desiredFrame           = -1;
outputXfer.bDisableExtraAudioInfo = true;

device.StopAutoCirculate (outputXfer.channelSpec);
device.InitAutoCirculate (outputXfer.channelSpec, /*startFrame*/ 0, /*endFrame*/ 6, /* # chls */ 1, NTV2_AUDIOSYSTEM_1);

outputTaskList.Clear ();
outputTaskList.AddTimeCodeReadTask ();
device.SetAutoCirculateCaptureTask (outputXfer.channelSpec, outputTaskList);
device.StartAutoCirculate (outputXfer.channelSpec);

while (!mGlobalQuit)
{
   AUTOCIRCULATE_STATUS_STRUCT acStatus;
   device.GetAutoCirculate (outputXfer.channelSpec, &acStatus);
   if (acStatus.state == NTV2_AUTOCIRCULATE_RUNNING && acStatus.bufferLevel < (ULWord)(acStatus.endFrame - acStatus.startFrame - 1))
   {
      RP188_STRUCT  tc;

      //  Output the timecode into the desired places...
      outputTaskList.Clear ();
      device.AddTimeCodeWriteTask (NULL, NULL, NULL, NULL, NULL, NULL, &tc, &tc, &tc);

      //   Transfer from the host to the device...
      device.TransferWithAutoCirculateEx2 (&outputXfer, &outputXferStatus);

   }   //   if A/C running and frame(s) are available for transfer
   else
      device.WaitForOutputVerticalInterrupt (NTV2_CHANNEL3);
} // loop til quit signaled

device.StopAutoCirculate (outputXfer.channelSpec);
. . .
```
### New API:
```
. . .
CNTV2Card               device (0);
AUTOCIRCULATE_TRANSFER  outputXfer;   // My A/C output transfer info 

device.AutoCirculateStop (NTV2_CHANNEL3);
device.AutoCirculateInitForOutput (NTV2_CHANNEL3, /*#frames*/ 7, NTV2_AUDIOSYSTEM_INVALID, AUTOCIRCULATE_WITH_RP188);
device.AutoCirculateStart (NTV2_CHANNEL3);

while (!mGlobalQuit)
{
   AUTOCIRCULATE_STATUS acStatus;

   device.AutoCirculateGetStatus (NTV2_CHANNEL3, acStatus);
   if (acStatus.IsRunning () && acStatus.CanAcceptMoreOutputFrames ())
   {
      NTV2_RP188 tc;
      outputXfer.SetOutputTimeCode (tc, NTV2_TCINDEX_SDI3);
      outputXfer.SetOutputTimeCode (tc, NTV2_TCINDEX_SDI4);
      outputXfer.SetOutputTimeCode (tc, NTV2_TCINDEX_SDI5);

      //  Transfer from the host to the device...
      device.AutoCirculateTransfer (NTV2_CHANNEL3, outputXfer);

   }   // if A/C running and frame is available for transfer
   else
      device.WaitForOutputVerticalInterrupt (NTV2_CHANNEL3);
} // loop til quit signaled

device.AutoCirculateStop (NTV2_CHANNEL3);
. . .
```
