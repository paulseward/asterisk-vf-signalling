# In-band Signalling for heritage equipment

These Asterisk Dialplan subroutines play back strings of tones associated with in-band VF (voice frequency) signalling protocols.  These can be used when routing calls through heritage equipment which responds to such signalling protocols.

## Credits
The design of these routines is heavily influenced by the `mfer` and `sfer` subroutines developed for NPSTN by Naveen Albert and Brian Clancy in 2018. https://npstn.us/docs/#mfer

I've restructured them to remove some of the redundant code, and expanded them to include DTMF and AC9 signaling.

## Installation
Put these files under `/etc/asterisk/signalling` and then include them by adding the following to the appropriate places in the dialplan:

```
[globals]
#include "/etc/asterisk/signalling/globals.conf"
```
and at the bottom of extensions.conf
```
#include "/etc/asterisk/signalling/send-vf.conf"
```

## Usage
```
GoSub(send-vf,s,1(sip-peer, sig-type, digits))

  sip-peer => SIP/peer-name
  sig-type => [mf|dtmf|ac9]
  digits   => [0-9a-d*#w] - not all digits make sense for all sig types

eg, to connect to the SIP peer 'my-fxo-port' then send 1234 as DTMF in-band:

GoSub(send-vf,s,1(SIP/my-fxo-port-1, dtmf, 1234))
```

Calling the signalling routines directly is also possible, if you don't need the bridge

```
GoSub(dtmfer,start,1(5551212)) ; Play DTMF tones for 5551212
GoSub(ac9er,start,1(5551212))  ; Play AC9 tones for 5551212
GoSub(mfer,start,1(5551212))   ; Play MF tones for 5551212
```

## FAQ
**Why not use SendDTMF()?**
No reason, other than consistency with the other signalling routines in here.  Also, this gives us a little more control over exactly how the tones are sent.

**Why not use Dial() with the M or U options?**
The `M` and `U` options to `Dial()` don't bridge the audio for the call until after the macro/subroutine returns.  That means that the caller doesn't get to hear the tones being sent.

**Why not compile a custom dialplan routine that calls `ast_dtmf_stream()` to inject the dialing sequence**
That appears to cause sync issued on T1 spans, requires custom code compilation, and this was easier!

**Why don't you support $signalling-method**
Mostly this supports signalling for equipment I interact with regularly.  If I don't use it, I haven't built it!

However, Pull Requests that add other in-band VF signalling types are welcome

**Does this work on FreePBX/other thing?**
I test/run this on Asterisk installed on Debian.  I don't have the resources to test on other platforms.
