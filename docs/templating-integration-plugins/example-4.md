---
sidebar_position: 5
---

# Example 4: Full Example
In this example, we will show a more complex method (taken from personal daily note command)

*****
In this example, I have a custom plugin `codedungeon.NotePlan` which contains a number of commands, one of which is called `cdWeekday` which I invoke each day in my NotePlan Daily Calendar Note.  There is quite a bit to this command, I will try to unpack every that is happening.

```js
// @flow

// import NPTemplating plugin
import NPTemplating from 'NPTemplating'

export async function cdWeekday(userDate: any = ''): Promise<void> {
  try {
    const content: string = Editor.content || ''
    Editor.insertTextAtCursor('Please wait...')

    let pivotDate = userDate.length === 0 ? await cdGetSelectedCalendarDate() : userDate
    pivotDate = moment(pivotDate).format('MM/DD/YYYY')

    // create some templateData (data and methods) that will be used within template
    const templateData = {
      data: {
        pivotDate,
        yesterday: moment(pivotDate).subtract(1, 'days').format('YYYY-MM-DD'),
        today: moment(pivotDate).format('YYYY-MM-DD'),
        tomorrow: moment(pivotDate).add(1, 'days').format('YYYY-MM-DD'),
      },
      methods: {
        handleRetrospect: async () => {
          if (new Date(pivotDate).getDay() === 5) {
            const retrospectNote = new DateModule().weekOf(moment(pivotDate).format('YYYY-MM-DD'))
            const retroData = { ...templateData, retroMeetingDate: templateData.data.today }
            await createRetrospectNote(retrospectNote, retroData)

            // $FlowFixMe
            return `\n#### 🗓 3:00 PM - 4:00 PM - Retrospect\n- [[${retrospectNote}]]\n`
          }
        },
      },
    }

   // invoke NPTemplating.renderTemplate method which loads template and processes
   // using `templateData`
    const result = await NPTemplating.renderTemplate('Weekday Overview', templateData)

    // create Standup Note (see Focused - 9:30 AM - 10:00 AM)
    await cdStandup(pivotDate)

    // This replaces the `Please wait...' status message
    Editor.replaceTextInCharacterRange(content + result, 0, 16384)

    // scrolls cursor to top of note
    Editor.highlightByIndex(0, 0)
  } catch (error) {
    cdLogError('cdWeekday', error)
  }
}

```

#### Weekday Overview Template

```markdown title="📋 Templates / 🧰 Dungeon / Weekday Overview"
# Weekday Overview
*****
# <%= date.format('ddd, MMM DD, YYYY', `${np.pivotDate}`) %>
*****
## 📚 Overview

> ☀️ <%- web.weather() %>
> 🙆 “<%- web.advice() %>”

*****
## 🎧 Focused

#### ☕️ 5:00 AM - 6:00 AM - Planning & Solitude
<%- Bible.votd() %>

#### 🧭 6:00 AM - 9:30 AM
- Follow-up Email, Slack, etc.
- Prepare for [[<%= date.format('YYYY-MM-DD', `${np.pivotDate}`) %> Standup]]

#### 🧭 9:30 AM - 10:00 AM - Standup
- [[<%= date.format('YYYY-MM-DD', `${np.pivotDate}`) %> Standup]]
> 🗣 “<%- web.affirmation() %>”

#### 🍴 1:00 PM - 2:00 PM - Mental Break / Lunch
🍱 Lunch
<%- await handleRetrospect() %>
#### 🏁 5:00 PM - 5:30 PM — Shutdown
📌 EOD Tasks

*****
## 🔖 Review
*Daily review, provide a summary of what was accomplished and what open items need to be carried over to tomorrow*

**📕 Tomorrow:** [[<%= np.tomorrow %>]]

```

Would produce the following output when the custom command is invoked on 2021-12-31

```markdown
# Fri, Dec 31, 2021
*****
## 📚 Overview

> ☀️ Fountain Valley, California, United States: ☀️ +58°F
> 🙆 “For every complex problem there is an answer that is clear, simple, and wrong.”

*****
## 🎧 Focused

#### ☕️ 5:00 AM - 6:00 AM - Planning & Solitude
> 🙏🏻  Numbers 7:34
> 🗣  one male goat for a purification offering;

#### 🧭 6:00 AM - 9:30 AM
- Prepare for [[2021-12-31 Standup]]

#### 🧭 9:30 AM - 10:00 AM - Standup
- [[2021-12-31 Standup]]
> 🗣 “You're a smart cookie”

#### 🗓 3:00 PM - 4:00 PM - Retrospect
- [[W52 (2021-12-26..2022-01-01)]]

*****
## 🔖 Review
**📕 Tomorrow:** [[2022-01-01]]
```

#### Template Breakdown
The breakdown of this template is as follows (for brevity, I have extracted only the lines which use template tags)

```markdown
...
3: # <%= date.format('ddd, MMM DD, YYYY', `${np.pivotDate}`) %>
...
7: > ☀️ <%- web.weather() %>
8: > 🙆 “<%- web.advice() %>”
...
14: <%- await Bible.votd() %>
...
21: * Prepare for [[<%= date.format('YYYY-MM-DD', `${np.pivotDate}`) %> Standup]]
22: > 🗣 “<%- web.affirmation() %>”
...
26: <%- await handleRetrospect() %>
...
34: **📕 Tomorrow:** [[<%= np.tomorrow %>]]
```

| Line        | Description |
| ----- | -------------------------------------------------------------------------------------------------------- |
| 3     | Displays date using `DateModule.format` method, and supply the date using `data.pivotDate` variable       |
| 7     | Displays current weather using `WebModule.weater` method       |
| 8     | Displays daily advice using `WebModule.advice` method      |
| 14    | Displays personal `Bible.votd` method (Bible Plugin , verse of the day)    |
|       | See [Templating Plugins](/docs/templating-plugins/overview) section for information on creating custom `np.Templating` Plugins    |
| 18    | Task to prepare for current day "Stand Up" (internal work function)    |
| 21    | Time Block: Link to current day "Stand Up" note    |
| 22    | Displays daily affirmation using `WebModule.affirmation` method    |
| 26    | Creates an additional time block entry for Friday retrospect, using `handleRetrospect` method (defined in templateData.methods)    |
| 34    | Daily review section, link to tomorrow date    |
