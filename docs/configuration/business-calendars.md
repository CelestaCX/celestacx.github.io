Here is the **Business Calendars** page:
 
---
 
## Business Calendars
 
Business calendars define when your operation is open, when it is closed, and how it handles public holidays and special schedules. CelestaCX uses calendars to control routing behaviour, queue availability, SLA calculations, and out-of-hours messaging. Every queue and many routing rules reference a calendar.
 
---
 
### How Calendars Are Used
 
Calendars are not just informational — they actively influence platform behaviour in real time:
 | Where  Calendars  Are  Used | Effect |
| --- | --- |
| Queue  availability | A  queue  linked  to  a  calendar  is  only  open  during  defined  working  hours.  Outside  those  hours,  overflow  rules  apply. |
| Routing  rules | Rules  with  a  time  condition  check  the  linked  calendar  before  deciding  whether  to  apply. |
| SLA  calculations | Elapsed  time  for  SLA  targets  is  measured  only  during  calendar  working  hours,  not  around  the  clock. |
| Out-of-hours  messaging | When  a  calendar  marks  a  queue  as  closed,  automated  out-of-hours  replies  are  triggered  on  digital  channels. |
| Reports | Historical  reports  can  filter  and  segment  interaction  data  by  calendar  periods. | 
---
 
### Calendar Components
 
A business calendar in CelestaCX consists of three components:
 | Component | Description |
| --- | --- |
| Working  Hours | The  standard  open  hours  for  each  day  of  the  week.  Each  day  can  have  different  hours,  or  be  marked  as  closed. |
| Holidays | Specific  dates  when  the  operation  is  closed  regardless  of  the  standard  weekly  schedule. |
| Special  Schedules | Date-specific  overrides  for  modified  hours  —  for  example,  reduced  hours  on  a  day  before  a  public  holiday,  or  extended  hours  during  a  campaign  period. | 
---
 
### Creating a Calendar
 
1. Navigate to **Administration → Business Calendars → Add Calendar** .
2. Enter a calendar name and optional description. Use a name that clearly identifies the scope — for example, "UK Operations — Standard" or "US East Coast — Support Team".
3. Set the **time zone** . All hours defined in this calendar are interpreted in the selected time zone. This is important for operations spanning multiple regions — create separate calendars per time zone rather than trying to manage offsets manually.
4. Define **Working Hours** for each day of the week:
 
- Set the open and close time for each day.
- Mark days as **Closed** if the operation does not run on that day.
- You can define a split schedule (e.g. a lunch break) by adding a second time block per day.
5. Add **Holidays** for the calendar year:
 
- Click **Add Holiday** , enter the date and a label (e.g. "Christmas Day").
- Holidays are treated as fully closed days. If you need reduced hours rather than full closure, use a Special Schedule instead.
6. Add any **Special Schedules** for known date-specific variations.
7. Save the calendar.
 
---
 
### Working Hours Configuration
 | Field | Description |
| --- | --- |
| Day | Day  of  the  week  (Monday  through  Sunday). |
| Open  Time | The  time  from  which  the  queue  or  rule  is  active. |
| Close  Time | The  time  at  which  the  queue  or  rule  becomes  inactive. |
| Closed | Toggle  to  mark  the  entire  day  as  closed. |
| Split  Hours | Optional  second  time  block  for  operations  with  a  mid-day  closure  (e.g.  09:00–12:30  and  13:30–17:30). | 
#### Example: Standard UK Business Hours
 | Day | Hours |
| --- | --- |
| Monday | 08:30  –  17:30 |
| Tuesday | 08:30  –  17:30 |
| Wednesday | 08:30  –  17:30 |
| Thursday | 08:30  –  17:30 |
| Friday | 08:30  –  17:00 |
| Saturday | Closed |
| Sunday | Closed | 
---
 
### Holidays
 
Holidays are stored as annual entries within each calendar. CelestaCX does not automatically import public holidays — they must be entered manually or imported via CSV.
 
#### Importing Holidays via CSV
 
To add a full year of public holidays at once:
 
1. Navigate to **Administration → Business Calendars → [Calendar] → Holidays → Import** .
2. Download the holiday import template.
3. Populate the CSV with columns: `date` (YYYY-MM-DD format) and `name` .
4. Upload the file. All valid rows are added immediately.
 **Tip:** Import holidays at the start of each calendar year, or whenever public holiday schedules are published. Set a recurring reminder to keep calendars accurate — incorrect holidays silently affect routing and SLA calculations. 
---
 
### Special Schedules
 
Special schedules override standard working hours for a specific date without marking the day as a full holiday. Common use cases include:
 
- Reduced hours on the day before a public holiday
- Extended hours during a promotional campaign or product launch
- Modified hours during staff training days
- A single-day change to open a normally closed day
 
To add a special schedule:
 
1. Navigate to **Administration → Business Calendars → [Calendar] → Special Schedules → Add** .
2. Select the date.
3. Set the modified open and close times, or mark the day as closed.
4. Add a label for internal reference (e.g. "Christmas Eve — Early Close").
5. Save. The special schedule takes precedence over the standard working hours for that date only.
 
---
 
### Time Zones & Multi-Region Operations
 
Each calendar is tied to a single time zone. For operations running across multiple regions or countries, the recommended approach is to create one calendar per operational region.
 | Scenario | Recommended  Approach |
| --- | --- |
| Single-region  operation | One  calendar  for  the  entire  operation. |
| Multi-region,  same  hours | One  calendar  per  region,  each  in  its  own  time  zone. |
| Multi-region,  different  hours | One  calendar  per  region  with  region-specific  working  hours  and  holidays. |
| 24/7  operation | Create  a  calendar  with  all  days  set  to  00:00–23:59.  No  holidays  unless  partial  closures  apply. |
| Follow-the-sun  support | Create  one  calendar  per  regional  shift  and  link  each  to  the  appropriate  queue. | **Important:** CelestaCX evaluates calendar rules using the time zone defined in the calendar, not the agent's local time zone or the server time zone. Always verify the calendar time zone is set correctly before going live. 
---
 
### Linking a Calendar to a Queue or Routing Rule
 
#### Linking to a Queue
 
1. Navigate to **Administration → Queues → [Queue] → Edit** .
2. In the **Business Calendar** field, select the appropriate calendar from the dropdown.
3. Save. The queue will now open and close according to the calendar schedule.
 
#### Linking to a Routing Rule
 
1. Navigate to **Administration → Routing → Routing Rules → [Rule] → Edit** .
2. In the **Time Condition** section, enable **Restrict to business hours** .
3. Select the calendar to reference.
4. Choose whether the rule applies **during open hours only** , **during closed hours only** , or **always** (calendar ignored).
5. Save.
 
---
 
### Testing a Calendar
 
Before linking a calendar to a live queue, verify its behaviour using the built-in calendar tester:
 
1. Navigate to **Administration → Business Calendars → [Calendar] → Test** .
2. Enter a date and time.
3. The tester will report whether that moment is within open hours, a holiday, or covered by a special schedule.
 
Use this to confirm edge cases — midnight transitions, daylight saving time boundaries, and holiday dates — before go-live.
 
---
 
### Calendar Best Practices
 
- **Name calendars clearly** with region and purpose included, especially in multi-tenant or multi-region deployments.
- **Set the correct time zone first** before configuring any hours. Changing the time zone after hours are set does not automatically adjust the times.
- **Review calendars annually** and update holidays at the start of each year.
- **Use special schedules** rather than temporary edits to working hours — they are date-scoped and revert automatically, reducing the risk of forgetting to restore normal hours.
- **Test before go-live** using the calendar tester, particularly for new deployments or after significant changes.
 
---
 
*What's next: Proceed to* [*Forms & Form Builder*](#) *to configure data-collection forms that agents use during and after interactions.*