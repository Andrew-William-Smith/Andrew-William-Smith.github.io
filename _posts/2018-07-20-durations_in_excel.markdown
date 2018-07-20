---
layout: post
title:  "Exhibition Schedules: Easily Working with Short Durations in Excel"
date:   2018-07-20 17:20:00
categories: acir msoffice
---

As part of our Summer Freestyle figure skating camp, the Allen Community Ice Rink hosts weekly exhibitions to aid skaters in preparing for the experience of performing their programs at competitions.  To optimise the running of these exhibitions, we often create schedules in Excel in order to track which skaters should be assigned to which warm-up sessions and whether participants have submitted musical accompaniment for their performances.  A key aspect of this scheduling system is timing: we must ensure that no warm-up group is so long that skaters will have to wait for an excessive time before performing their programs, and we must ensure that each exhibition as a whole does not exceed its alotted time slot.  In this article, I describe a procedure by which I facilitate working with short durations of time (`minutes:seconds`), such as figure skating program lengths, in Microsoft Excel.

# Entering durations
To enter a duration comprised of minutes and seconds, simply enter the number of minutes and seconds, separated by a colon, in any cell.  Thus, one minute and thirty seconds would be entered `1:30`, and three minutes and seventeen seconds would be `3:17`.  After you hit the <span class="key">Enter</span> key to commit the duration to the cell, you may notice some odd behaviour: if you look at the formula bar, the duration we have just entered appears as a regular time, with our minutes being shown as hours.  Furthermore, if you have a duration with more than 23 minutes, the minutes will roll back around to `00`.  Examples of this formatting are shown below:

| Duration entered | Duration displayed |
| :--------------: | :----------------: |
| 1:30  | 01:30 |
| 3:17  | 03:17 |
| 24:32 | 00:32 |

The reason for this strange display is that, by entering our durations in the format that we did, we saved ourselves from having to type a leading `0:` before each; however, Excel now treats our minutes as hours in a regular time of the day, and our seconds as minutes.  To fix this anomaly, we can take advantage of custom cell formats.

# 525,600 minutes: reformatting durations
To show our entered minutes as minutes, we must reformat the cells in which we entered them.  Select all cells containing durations, then click the small arrow in the bottom-right corner of the “Number” section of the ”Home” tab on the Ribbon.

![Custom formatting button](/assets/2018-07-20/customformat.png){:.fullwidth}

In the “Format Cells” window that appears, ensure that the “Custom” category is selected in the menu at the right.  In the text box under the “Type” header, enter `[h]:mm`, then click “OK”.

![Custom duration format](/assets/2018-07-20/formatstr.png){:.sm-center}

All durations you selected should now be displayed with their full minute lengths (the `[h]` in the custom format), followed by their seconds (the `mm`).  While working with durations in this way is quite simple in isolation, adding them to actual times can be somewhat difficult; if such functionality is required, it is recommended to enter durations using a leading zero (e.g. `00:03:17`) or via the `TIME` function.  Nevertheless, for quick checks of group lengths and event durations, this workflow is quite efficient and requires much less typing than “proper” time strings.
