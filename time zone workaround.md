- That's actually a clever workaround.
  
  By setting the server's reference time zone to **America/Guatemala**, you're effectively using a time zone that:
	- Remains at **UTC-6 all year**.
	- Does **not observe DST**.
	- Matches the current behavior of **Mexico City** after Mexico abolished DST in most regions.
- In practice:
  ```
  Actual desired time:
  UTC-6 year-round
  
  Server:
  America/Guatemala
  UTC-6 year-round
  
  Old Client:
  Mexico (obsolete tzdata)
  Applies DST incorrectly
  
  After sync:
  Client receives configuration based on America/Guatemala
  No DST transitions occur
  
  Result:
  Server and client remain aligned
  ```
- ### Why it works
  
  The client's problem is not the clock itself but the **DST rule associated with the time zone identifier**.
  For example, an outdated client may interpret:
  
  `America/Mexico_City`
  as:
  ```
  UTC-6 (winter)
  UTC-5 (summer)
  ```
  
  even though that is no longer correct.
  
  By switching to:
  
  `America/Guatemala`
  
  the old tzdata already knows that Guatemala does not use DST, so there is no seasonal transition to apply.
- ### Advantages
	- No OS update required on legacy clients.
	- Prevents the March-November one-hour drift.
	- Keeps local time at UTC-6 all year.
	- Simple to deploy if the server pushes the time zone to clients.
-
- ### Potential Caveat
  
  From a pure time zone semantics perspective, the client is no longer configured with its true geographical time zone.
  
  So:
  ```
  Location: Mexico City
  Configured TZ: America/Guatemala
  ```
  This is technically inaccurate, even though the UTC offset behavior is currently identical.
  
  If Mexico's time policy changes again in the future, or if an application checks the actual time zone name rather than the offset, you may need to revisit the workaround.
- ### How I would document it
  
  > Due to legacy clients running obsolete timezone definitions, Mexico DST rules are incorrectly applied between March and November, producing a one-hour offset. As a mitigation, the server reference timezone was changed from Mexico City to America/Guatemala (UTC-6, no DST). Because both locations currently share the same UTC offset year-round, this prevents erroneous DST adjustments on legacy clients and maintains synchronization with the server.
  
  I would classify this as a **compatibility workaround** rather than a true fix. The true fix remains updating the client OS tzdata, but your approach is a practical way to keep older systems synchronized.
  
  <!--EndFragment-->