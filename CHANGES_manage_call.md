# Pragon PA: "Manage the call"

New ring option next to OK / ignore / reject. Say **manage** (or tap **MANAGE** on the ring notification).

## Flow
1. Pragon answers (loudspeaker, same as OK), introduces itself and asks who / why / urgency / callback.
2. Each time it learns something (`save_call_info`) the ongoing notification updates: *"Ravi · delivery problem · urgency: high"*.
3. Caller asks for you -> Pragon says "please hold" and calls `ask_owner`. Your phone vibrates (no sound, the caller would hear it)
   and a heads-up shows **CONNECT** / **I'M BUSY**. Voice also works: "connect him", "tell him I'm busy" (or "Pragon connect" to the live call).
4. **CONNECT** -> Pragon says "connecting you now", then audio goes back to the earpiece and you are on the call.
   **I'M BUSY** -> Pragon tells the caller you're busy, takes a message / callback, says goodbye and hangs up.
   No answer in 30 s = busy. Safety nets: 12 s after CONNECT the call is handed over anyway; 45 s after BUSY the call is cut.
5. After the call: the usual spoken summary + notification, now also saying "They asked to speak to you" and what you decided.

## Files
Changed: `PragonPA.kt` (flow, prompt, notifications, timers), `LiveSession.kt` (`managed`, `tell()`, tools `ask_owner` / `connect_owner`),
`V9Voice.kt` (manage / connect / busy phrases), `PragonContacts.kt` + `Tools.kt` (`pa connect|busy`), `AndroidManifest.xml` (VIBRATE),
`assets/settings.html` (help text), `tests/v9voice_regex_check.py` (new cases, 49 pass).

## NOT verified: please read
No Kotlin compiler or Android SDK was available, so this has **not been compiled or run on a phone**. Parser regexes are tested
(python mirror). Expect to fix a compile error or two, then test with a second phone calling yours:
1. Turn on Pragon PA, call yourself, say "manage": notification should read "Pragon is managing the call...".
2. Say you want to speak to the owner: vibration + CONNECT / I'M BUSY. Test each button, the voice phrases, and 30 s silence.

Known limits: Android can't route audio privately during a call, so Pragon never *speaks* the alert to you (the caller would hear it);
it uses vibration + notification. Some phones silence the microphone during calls and can't run the assistant part at all.
The "ok" button was replaced by MANAGE in the notification (Android shows 3 buttons); saying "OK" still works.

## Update: Hear Pragon + "speaker"
- **Settings > Pragon PA > "Hear Pragon on calls"** (ON by default). ON = Pragon talks on the loudspeaker: you hear caller + Pragon, and the
  phone's microphone can carry Pragon's voice to the caller. OFF = Pragon talks through the earpiece: you hear only the caller.
- **Say "speaker"** at the ring prompt: answers the call and puts it on the loudspeaker whatever the setting says. "Manage on speaker" does both.
- Why the caller may not have heard Pragon before: the route can be reset when the call connects and a quiet speaker isn't picked up by the mic.
  Now the speaker route is re-applied for the first ~12 s and the call volume is set to max for the call (restored when it ends / you take over).
- Android apps cannot inject audio into a cellular call; the caller can only hear Pragon through the phone's microphone. If a phone still
  doesn't carry it, that phone's call audio is blocking it and nothing in the app can change that.
