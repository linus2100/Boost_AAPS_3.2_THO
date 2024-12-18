# AndroidAPS - Boost #
* Check the wiki: https://androidaps.readthedocs.io
*  Everyone who’s been looping with AndroidAPS needs to fill out the form after 3 days of looping  https://docs.google.com/forms/d/14KcMjlINPMJHVt28MDRupa4sz4DDIooI4SrW0P3HSN8/viewform?c=0&w=1

[![Support Server](https://img.shields.io/discord/629952586895851530.svg?label=Discord&logo=Discord&colorB=7289da&style=for-the-badge)](https://discord.gg/4fQUWHZ4Mw)

[![CircleCI](https://circleci.com/gh/nightscout/AndroidAPS/tree/master.svg?style=svg)](https://circleci.com/gh/nightscout/AndroidAPS/tree/master)
[![Crowdin](https://d322cqt584bo4o.cloudfront.net/androidaps/localized.svg)](https://translations.androidaps.org/project/androidaps)
[![Documentation Status](https://readthedocs.org/projects/androidaps/badge/?version=latest)](https://androidaps.readthedocs.io/en/latest/?badge=latest)
[![codecov](https://codecov.io/gh/nightscout/AndroidAPS/branch/master/graph/badge.svg)](https://codecov.io/gh/MilosKozak/AndroidAPS)

DEV: 
[![CircleCI](https://circleci.com/gh/nightscout/AndroidAPS/tree/dev.svg?style=svg)](https://circleci.com/gh/nightscout/AndroidAPS/tree/dev)
[![codecov](https://codecov.io/gh/nightscout/AndroidAPS/branch/dev/graph/badge.svg)](https://codecov.io/gh/MilosKozak/AndroidAPS)



***Boost 4.1.5 based on AAPS Master 3.2.0.1***

Boost has been updated to the 3.2.0.1 codebase. This means that it requires Nightscout 15 and updates your database. Once you have enabled NS15 and elected to use the NSClient V3 to interact with it, you can do a full synchronisation with the server (go to NSV3 tab, select the three dots to the top right and choose "Full Synchronisation"), which will pull enough data into app to run DynamiISF.

Additionally, we have made a number of modifications. Please read the Start and End time section, as this may require a change to stop Boost crashing. 

**Dynamic ISF**

Dynamic ISF settings have been removed from the Boost settings and are now a separate item in the Absorption Settings preference. Within this, there is a switch to enable or disable Dynamic ISF. If disabled, your profile ISF will be used. There is also the feature to set the glucose cap level, at which point ISF values decrease at a lower rate as glucose levels continue to increase. The DynISF adjustment factor is also in these settings, along with a switch to enable the choice of whether profile switches are obeyed.
  
Traditional Autosens is deprecated in this code and sensitivityRatio is calculated using 'Eight hour weighted average TDD  / 7-day average TDD', if the "Adjust sensitivity and BG option" is selected.

Boost uses a similar version of DynamicISF for making predictions, however, unlike the hardcoded quanta for the different values of insulin peak, when free-peak is used, it scales between the highest and lowest values.

The ISF for dosing decisions within Boost is slighty different to that in DynamicISF. The calculation is intended to mimic the effects of higher insulin sensitivty at lower glucose levels, and runs as follows:

1. With COB and increasing deltas, use 75% of the predicted BG and 25% of the current BG.
2. If current BG is accelerating fast, bg is below 180mg/dl/10mmol/l and eventual BG is higher than current, use 50% of both eventual and current BG.
3. If BG is between 160 and 270 and almost flat, and eventual BG is lower than BG, use 60% min predicted BG and 40% current BG.
4. If BG is increasing slowly, and above 198 or eventual BG is above current BG and current BG is above 198,  use 40% min predicted BG and 60% current BG.
5. If BG is increasing more slowy, or eventual BG is greater than current BG, use current BG.
6. If BG is not increasing, use minimum predicted BG. 

The user adjustable Low Glucose Suspend threshold has reverted to the Boost preferences pane. This allows the user to set a value higher than the system would normally use, such that when predictions drop below this level, a zero TBR is set.

**Night Mode**

We have implemented an option called Night Mode in the Treatments Safety pane. This enables SMBs to be disabled overnight in certain circumstances. The settings are:

*BG Offset:* When Night Mode is enabled, this is the value above your target at which point SMBs will be re-enabled.
*Start and end times:* Allow you to choose when this function is active.
*Disable with COB:* Disables Night Mode if there are COB.
**Disable with low temporary target:* If a low temp target has been set, SMBs can be enabled.

**Boost start and end times**

The start and end times ahve been adapted and now use a 24 hour clock. You will need to format this in an hh:mm format, and its possible that in some cases, if you dont change what youve updated from, Boost will crash. 

The end time can now run over midnight, so you can set a start time of 07:00 and an end time of 02:00.

***Boost***

You can use Boost when announcing carbs or without announcing carbs. With COB there is an additional piece of bolusing code that oeprates for the first 40 mins of COB. If you prefer to manually bolus, it fully supports that with no other code.

It also has variable insulin percentage determined by the user, and while boost time is valid, the algorithm can bolus up to a maximum bolus defined by the user in preferences.

The intention of this code is to deliver an early, larger bolus when rises are detected to intiate UAM deviations and to allow the algorithm to be more aggressive. Other than Boost, it relies on oref1 adjusted to use the variable ISF function based on TDD.

All of the additional code outside of the standard SMB calculation requires a time period to be specified within which it is active. The default time settings disable the code. The time period is specified in hours using a 24 hour clock in the Boost preferences section.

**COB:** ***Note: Boost is not designed to be used with eCarbs. This may result in additional, unexpected bolusing. Do not use it.***

With Carbs on Board, Boost has a 15 minute window to deliver the equivalent of a mealtime bolus and **is allowed to go higher than your Boost Bolus Cap**, up to `InsulinRequired/insulin required percent` calculated by the oref1 algorithm, taking carbs into account. In the following 40 mins after the carbs are added, it can do additional larger boluses, as long as there is a delta >5 and COB > 0. The max allowed is the greater of the Boost Bolus Cap or the "COB cap", which is calculated as `COB / Carb Ratio`.

During normal use, you should set your Boost Bolus Cap to be the max that boost delivers when Boost is enabled and no COB are entered.

Boost outside the first 40 mins of COB, or with 0 COB has six phases:

1. Boost bolus
2. High Boost Bolus
3. Percentage Scale
4. Accleration Bolus
5. Enhanced oref1
6. Regular oref1

**Boost**

When an initial rise is detected with a meal, but no announced COB, delta, short_avgDelta and long_avgDelta are used to trigger the early bolus (assuming IOB is below a user defined amount). The early bolus value is one hour of basal requirement and is based on the current period basal rate, unless this is smaller than "Insulin Required" when that is used instead. This only works between 5mmol/l (90mg/dl) and 10mmol/l (180mg/dl)

The user defined Boost Scale Value can be used to increase the boost bolus if the user requires, however, users should be aware that this increases the risk of hypos when small rises occur.

Boost also uses the percent scale value to increase the early bolus size.

If **Boost Scale Value** is less than 3, Boost is enabled.

The short and long average delta clauses disable boost once delta and the average deltas are aligned. There is a preferences setting (Boost Bolus Cap) that limits the maximum bolus that can be delivered by Boost outside of the standard UAMSMBBasalMinutes limit.

**High Boost**

If glucose levels are above 10, and glucose acceleration is greater than 5%, a high boost is delivered. The bolus value is one hour of basal requirement and is based on the current period basal rate, unless this is smaller than "Insulin Required" when one hour of basal plus half the insulin required is used, divided by your "pecentage of insulin required value", unless this value is more than insulin required, at which point that is used.

**Boost Percentage Scale**

Boost percentage Scale is a feature that allows Boost to scale the SMB from a user entered multiple of insulin required at 108 mg/dl (6 mmol/l) to the user entered *Boost insulin required percent* at 180mg/dl (10 mmol/l). It can be enabled via a switch in the preferences and the percentage values are hard coded. it is only active when [Delta - Short Average Delta ] is positive, meaning that it only happens when delta variation is accelerating.

**Acceleration bolus**

The accleration bolus is used when glucose levels are rising very rapidly (more than 25%) when a dose that is scaled similar to the Percent Scale is used, with the scaling operating at half the rate of the "Boost Percentage Scale" option.

**Enhanced oref1**

If none of the above conditions are met, standard SMB logic is used to size SMBs, with the insulin required PCT entered in preferences. This only works on positive deviations and similar to the percent scale, when deltas are getting larger. Enhanced oref1 uses regular insulin sizing logic but can dose up to the Boost Bolus cap.

**Regular oref1**

Once you are outside the Boost hours, "max minutes of basal to limit SMB to for UAM" is enabled, and the dosing works in the same way as regular OpenAPSSMB.

With Boost and Percent Scale functions, the algorithm can set a 5x current basal rate in this run of the algorithm, with a cap of 2x insulin required, as per normal oref1. This is reassesed at each glucose point. 

Enable Boost with High Temp Target is carried through. This allows Boost, Percent Scale and Enhanced oref1 to be disabled when a user sets a high temp target, while retaining SMBs.

Enhanced oref1 only fires when deltas are increasing above a rate of 0.5%. This reduces the amount of times it fires when glucose levels are higher, but still allows additional bolusing.

The **BOOST** settings have a number of extra items:

Note that the default settings are designed to disable most of the functions, and you will need to adjust them.

*Boost insulin required percent* - defaults to 50% can be increased, but increasing increases hypo risk.<br>
*Boost Scale Value* - defaults to 1.0. Only increase multiplier once you have trialled. <br>
*Boost Bolus Cap* - defaults to 0.1 <br>
*Percent scale factor* - defaults to 200 <br>
*UAM Boost max IOB* - defaults to 0.1 <br>
*UAM Boost Start Time (using 24 hour clock)* - needs to be set using 24hr clocknformat, e.g. 00:30. Defaults to 7<br>
*UAM Boost end time (using 24 hour clock)* - needs to be set using 24hr clocknformat, e.g. 00:30. Defaults to 8<br>
*BG level below which low glucose suspend occurs* - defaults to standard algorithm behaviour; can be adjusted between 65 and 100.<br>

**Notes on settings**

The settings with the largest effect on post prandial outcomes are *Boost insulin required percent* and *Percent Scale Factor*, alongside your usual SMBMinutes settings.

*Boost insulin required percent* under normal AAPS, percentage of insulin required to be given as SMB is hardcoded to 50%. This setting allows you to tell the accelerated dosing features to give more a higher percentage of the insulin required. <br>
*Percent scale factor* is the max amount that the Boost and Percent Scale functions can multiply the insulin required by at lower glucose levels. A larger number here leads to more insulin. <br>
SMBMinutes settings: When there is no longer any acceleration in glucose delta values, the algorithm reverts to standard oref1 code and uses SMBminutes values as its max SMB size. When using Boost these values should generally be set to less then the default 30mins. A max of 15 or 20 is usually best.

**Recommended Settings**

*Boost Bolus Cap* - Start at 2.5% of TDD and increase to no more than 5% of 7 day average total daily dose. <br>
*Percent scale factor* - Once you are familiar with the percentage scale factor, the values can be increased up to 500% with associated increase in hypo risk with rises that are not linked to food.
*UAM Boost max IOB* - Start at 5% of TDD and increase to no more than 15% of 7 day average total daily dose. <br>
*UAMSMBBasalMinutes* - 20 mins. This is only used overnight when IOB is large enough to trigger UAM, so it doesn't need to be a large value. <br>
*Boost insulin required percent* - recommended not to exceed 75%. Start at 50% and increase as necessary. <br>
*Target* - Set a target of 6.5mmol/l (120mg/dl) to get started with Boost. This provides a cushion as you adjust settings. Values below 5.5mmol/l (100mg/DL) are not recommended.<br>

**Stepcount Features**

The three stepcount features in *Boost 4.1.5* are:

1. *Inactivity Detection* Inactivity detection determines when the stepcount is below a user defined limit over the previous hour, and increases basal and DynamicISF adjustment factor by a user defined percentage. The defaults are 400 steps and increase to 130%. Inactivity detection does not work when Sleep-in protection is active.
2. *Sleep-in protection* Sleep-in protection checks stepcount for a user defined period (in hours) after the Boost start time, and if it is below a user defined threshold, extends the time during which Boost and Percent Scale are disabled. The defaults are 2 hours and 250 steps. The maximum value for this is now 18 hours. Inactivity detection doesn't work while sleep-in is active.
3. *Activity detection* Activity detection allows a user to set the number of steps in the past 5 mins, 30 mins and hour as triggers for activity. If any of these are true, it will set a user defined lower percentage, to reduce basal and dynamicISF adjustment factor.  For the five minute setting, it will wait for 15 mins to revert to non-activity. The other two settings wait for the value for the period to drop below the threshold. The defaults are 420 steps for 5 mins (which corresponds to the 5 minute activity trigger on a Garmin), 1200 for 30 mins and 1800 for 60 mins. Profile decrease is set to 80%. 

Both activity detection settings are overridden by a percentage profile switch.

There are no enable/disable buttons for these settings, however, in both activity detection settings, *if the % value is set to 100, they have no effect*. Similarly, *if the Sleep-in protection hours are set to 0, it has no effect*.

**Libre Users**

If you are a Libre user, there is a setting in the DynamicISF preferences (you will have to switch to the DynamicISF plugin to set it) called "Enable SMB disregarding of BG source (UNSAFE)". This switch allows SMBs always with Libre devices across all APS plugins. If you choose to do this, please make sure you are using a sensor collection app that is providing glucose data every five minutes, and enable at least the Average Smoothing plugin in AAPS 3.2.

