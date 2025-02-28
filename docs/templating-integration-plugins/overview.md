---
sidebar_position: 1
---

# Overview
While `np.Templating` includes a number of commands for NotePlan users, the real power of `np.Templating` is the ability to integrate into custom NotePlan plugins.

## Templating Configuration
`np.Templating` has a suite of methods which can be used in your custom NotePlan plugins.

### Templating Methods
The following methods are available in the `NPTemplating` plugin

xxxx

### Integrating into NotePlan plugin

- Import NPTemplating Plugin
- Setup NPTemplating data object
-
#### Import NPTemplating Plugin
The first step, you will need to `import` the `NPTemplating` plugin into your source file

```javascript
import NPTemplating from "NPTemplating"
```

#### Setup NPTemplating data object
The next setup will be to define a templateData object
:::note
If you do not intend to include custom variables or methods, this step can be ignored
:::

```javascript
const templateData = {
      data: {/* your data object goes here */},
      methods: {/* your method object here */}
    }
```

For example, if your `templateData` object contains two variables (data) and two methods, it would look like this

```javascript
const templateData = {
      data: {
        fname: 'Mike',
        lname: 'Erickson'
			},
      methods: {
        uppercase: (str : string = '') => {
          return str.toUpperCase()
        },
        lowercase: (str : string = '') => {
          return str.toLowerCase()
         }
       }
    }
```

#### Invoke `NPTemplating.renderTemplate` method
The next step (yes, it is that easy) is to call the `NPTemplating.renderTemplate` method which will load the `templateName` and pass in the `templateData` object

```javascript
const result = await NPTemplating.renderTemplate('templateName', templateData)
```

#### Insert result into NotePlan Note
The final step, insert the result into NotePlan note (project or calendar note)

```javascript
Editor.insertTextAtCursor(result)
```

## Upgrading from `nmn.Templates`
The legacy NotePlan templating plugin, `nmn.Templates` does this a bit differently from `np.Templating`.

The following information will help you migrate from `nmn.Templates` to `np.Templating`

### Update `dailyStart` method
In this example, we will refactor the `jgClark.DailyJournal.dailyStart` command.

```javascript title="jgClark.DailyJournal.dailyStart"
// Start the currently open daily note with the user's Daily Note Template
export async function dayStart(today: boolean = false) {
  console.log(`\ndayStart:`)
  if (today) {
    // open today's date in the main window, and read content
    await Editor.openNoteByDate(new Date(), false)
    // $FlowIgnore[incompatible-call]
    console.log(`Opened: ${displayTitle(Editor.note)}`)
  } else {
    // apply daily template in the currently open daily note
    if (Editor.note == null || Editor.type !== 'Calendar') {
      await showMessage('Please run again with a calendar note open.')
      return
    }
  }
  await applyNamedTemplate(pref_templateTitle)
}
```

#### Modified `jgClark.DailyJournal.dailyStart` method
The following is the modified method (only modified 3 lines of code)

- import `NPTemplating` plugin
- Remove `applyNamedTemplate` method
- Call `NPTemplating.renderTemplate` method
- Insert result to current cursor in NotePlan note

```javascript title="jgClark.DailyJournal.dailyStart"
...
import NPTemplating from "NPTemplating"

// Start the currently open daily note with the user's Daily Note Template
export async function dayStart(today: boolean = false) {
  ...
	const result = await NPTemplating.renderTemplate('Daily Note Template', templateData)
	Editor.insertTextAtCursor(result)
    // await applyNamedTemplate(pref_templateTitle)
  ...
}
...
```

### Add Template Methods
When using `nmn.Templates` there is an `addTag` method which is included in the `nmn.Templates.templateController/src/processTemplate` method. You can use the `np.Templating.globals` method, which provides a mapping to current `nmn.Templates.addTag` method.
:::info
At time of release, all existing global methods which have been added to `nmn.Templates.templateController` using `addTag` method have been added to `np.Templating/lib/globals.js`
:::

### Daily Note Template Modification
The final step will be to modify the existing `Daily Note Template` to reference `np.Templating` variables and methods.

```markdown title="📋 Templates / Daily Note Template"
# Extended Daily Note Template
---

<%-formattedDateTime({format: '%A, %B %d, %Y'}) %>

## Today's events:
{{listTodaysEvents({template:"- *|START|*-*|END|*: *|TITLE|*",allday_template:"- *|TITLE|*"})}}

---

## Tasks
{{sweepTasks({limit:{ "unit": "day", "num": 7 },includeHeadings:false, ignoreFolders:['📋 Templates','_TEST','zDELETEME']})}}
---
## Notes


---
{{weather()}}

### Things to think about:
- "{{affirmation()}}."
- "{{advice()}}"
- {{quote()}}
```

## Examples
In the following examples, you can get a feel for how to integrate `np.Templating` into your NotePlan Plugins

- [Example 1](/docs/templating-integration-plugins/example-1): A simple "Hello World" type example
- [Example 2](/docs/templating-integration-plugins/example-2): Demonstrates how to use your plugin variables in a `np.Templating` template
- [Example 3](/docs/templating-integration-plugins/example-3): Demonstrates how to use your plugin methods in a `np.Templating` template
- [Example 4](/docs/templating-integration-plugins/example-4): Demonstrates a more complex example, showing how to use your own variables and methods
