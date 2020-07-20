---
title: "New UIDatePicker in iOS 14"
date: 2020-07-08T07:00:00-04:00
date: 2020-07-08T07:00:00-04:00
originalDate: 2020-07-05T21:48:43-04:00
publishDate: 2020-07-08T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - uikit
 - uidatepicker
 - swift
 - programming
 - apple
 - ios
 - ipados
 - wwdc2020
 - apple
categories:
 - development
description: "Learn about the new UIDatePicker in iOS 14."
keywords:
 - uikit
 - uidatepicker
 - swift
 - ios
 - tvos
 - ipados
 - wwdc2020
 - apple
---

WWDC2020 brought many interesting and unexpected updates to many old and well known APIs. In this article, we will explore what's new with `UIDatePicker` on iOS, an API that has existed since the dawn of time and hasn't changed much since its introduction.

# A Short History on Pickers

`UIDatePicker` is an API that has existed since the very early days of the iOS SDK - it goes all the way back to iOS 2.0.

If you have been developing for iOS for a while, you must remember this beauty:

![The Very First UIDatePicker](/img/date_picker_pre_7.png)

In iOS 7, the whole system received a full redesign. Look at the *huge* update our old friend `UIDatePicker` went through:

![The Second Date Picker](/img/date_picker_post_7.png)

... Not much of a chance for our old friend. But iOS 14 introduced a whole new date picker we can use. It's much easier and flexible to use for our end users, and we don't have to do much to adopt it.

# UIDatePicker in iOS 14

## UIDatePicker Styles

First things first, the old wheel-styled picker is not actually gone. Instead, `UIDatePicker` now has a property called `datePickerStyle` where you can let the system choose the best style with the `.automatic` style, or you can choose between `.compact` and `.inline` - both new to iOS 14 - or `.wheel`, which is the old style we have known for over a decade.

### The .compact Style

The `.compact` date picker style presents the user as a small UI the user can tap.

![Compact Style Calendar](/img/date_picker_ios14_entry.png)

This tiny UI takes up less space, and the best part is it is interactive. When the user taps it, they will view the new full calendar view in all its glory:

![Full Calendar](/img/date_picker_ios14_calendar.png)

In this full calendar view, your user has more flexibility to choose a time and a date, with arrows to move between months and more.

The picker is still as customizable as always. You can for example, show a prompt to select only the date or only the time instead of both as it is by default.

![Showing only the Time component of a picker](/img/picker_time_only.png)

![Showing only the Time component of a picker - editable](/img/picker_time_only_displayed.png)

### The .inline Style

This style is essentially the same as `.compact` with the difference that your user will never see a little UI they have to tap. Instead, the calendar or time picker component will be there in all its glory ready to be used.

![Inline UIDatePicker](/img/inline_ui_date_picker.png)

Finally, in either `.inline` or `.compact` modes, your user can tap the top right label that shows the month and year and the system will show a wheel picker to let them quickly jump to a different month and year:

![Quickly Jump to Months and Year](/img/inline_uidatepicker_year_month.png)

# Conclusion

Old and known APIs have received a deserved refresh in iOS 14. The new date picker is easy to implement and it provides your users with a faster way to check dates.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.