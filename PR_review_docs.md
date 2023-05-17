## PR review docs

### Aproach and general rules

#### Pull/Merge Requests

Main branches like master and develop should be protected and code changes should enter these branches only via pull/merge requests.

If developer works on a feature branch for more than a few hours, it is good to share the intermediate results with the rest of the team. To do this, create a pull/merge request without assigning it to anyone. Instead, mention people in the description or a comment and open the PR as a Draft PR. This indicates that the pull/merge request is not ready to be merged yet, but feedback is welcome.

Team members can comment on the pull/merge request in general or on specific lines with line comments. The pull/merge request serves as a code review tool, and no separate code review tools should be needed. If the review reveals shortcomings, anyone can commit and push a fix. Usually, the person to do this is the creator of the pull/merge request. The diff in the pull/merge request automatically updates when new commits are pushed to the branch.

When you are ready for your feature branch to be merged, assign the pull/merge request to the person who knows most about the codebase you are changing. Also, mention any other people from whom you would like feedback. After the assigned person feels comfortable with the result, they can merge the branch. If the assigned person does not feel comfortable, they can request more changes or close the pull/merge request without merging.

After you merge a feature branch, you should remove it from the source control software. Removing finished branches ensures that the list of branches shows only work in progress.

#### Conducting a Code Review

For a productive and effective code review, follow below best practice advices.

##### Know What to Look for

Know what to look for in code reviews: structure, style, logic, performance. readability (and maintainability), etc.

Reviewing code with certain questions in mind can help you focus on the right things. For instance, you might evaluate code to answer:

- Do I understand what the code does?
- Does the code function as I expect it to?
- Does this code fulfil regulatory requirements?

##### Review Fewer than 400 Lines of Code at a Time

A study revealed that developers should review no more than 200 to 400 lines of code (LOC) at a time.

In practice, a review of 200-400 LOC over 60 to 90 minutes should yield 70-90% defect discovery.

##### Inspection rates should under 500 LOC per hour

Research shows a significant drop in defect density at rates faster than 500 LOC per hour. Code reviews in reasonable quantity, at a slower pace for a limited amount of time results in the most effective code review.

##### Create Positive Review Culture
Try to be constructive in your feedback, rather than critical. You can do this by asking questions, rather than making statements.

___
### Code quality rules

A list of questions to ask ourselves when we are reviewing code. These can also be used when coding, and when doing a self-review before opening a PR.

---
- can the code be simpler
  - often when we write code it's hard to take a step back and see that we made something more complex than it needs to be. But reviewing code (even our own at the end of a task) it's easier to spot. This can be something small (like nesting multiple ternary operators when a `switch` could have worked) or something bigger (trying to solve multiple use cases with one component, making it overly complicated)
  - when we can't understand why something is written in a certain way, it might be an indicator that it's more complicated than it needs to be
___
- should this code be in a component or can it be placed outside?
  - any function that doesn't depend on component state can (and should) be moved outside the component
```typescript
const UIComponent: React.FC = () => {
  const { company } = useAppSelector(state => state.company.company)

  // this function should be outside the component
  const getCompanyInitials = (name: string): string => {
    return name.split(' ').map((n) => n[0]).toUpperCase().join('')
  }
}
```
- CSS
  - where we can we should use shorthand properties:
```css
.container {
  padding: 20px 15px 20px 15px;
  border-width: 1px;
  border-style: solid;
  border-color: #000;
}
```
```css
.container {
	padding: 20px 15px;
	border: 1px solid #000;
}
```
description of shorthand properties: [mdn docs](https://developer.mozilla.org/en-US/docs/Web/CSS/Shorthand_properties)
  - always prefer flex and gap instead of horizontal and vertical margins between items. As a good rule of thumb, we use margin and padding for the container, and then set `display: flex` for the items inside the container, and use flex properties to position the items. Using `gap` means the last item (row or columns) won't have a margin, gap only adds space _between_ the flex items.
  - should a color value be a hardcoded hex code or an imported variable?
  - since we are using SCSS, we should properly place classes inside their parent classes:
```css
.container {
	padding: 20px 15px;
	display: flex;
	gap: 5px;
}
.item {
	background-color: #ccc;
	border-radius: 5px;
}

```
```scss
.container {
	padding: 20px 15px;
	display: flex;
	gap: 5px;

	.item {
		background-color: $color_gray;
		border-radius: 5px;
	}
}
```
___
- can the code be more strictly typed?
  - one of the more common things i notice is using string as a type where an enum would be better to enforce better type checking
```typescript
const getFooLabel = (fooType: string): string => {
  if (fooType === 'dog') {
    return 'bark'
  } else if (fooType === 'cat') {
    return 'meow'
  } else if (fooType === 'cow') {
    return 'moo'
  } else {
    return ''
  }
}
```
correct code:
```typescript
enum FooType {
  Dog = 'dog',
  Cat = 'cat',
  Cow = 'cow'
}
const getFooLabel = (fooType: FooType): string => {
  switch (FooType) {
    case (FooType.Dog):
      return 'bark'
    case (FooType.Cat):
      return 'meow'
    case (FooType.Cow):
      return 'moo'
    default:
      return ''
  }
}
```
it seems like it's more code but we can most likely use this `FooType` enum in multiple places, to tell Typescript to only expect these specific values. Also notice the use of `switch ` instead of `if` - switch is great when dealing with a strictly known set of values.
___
- can some code be exported into a utility function?
  - if you notice a line (or several) of code that performs some calculation or determines a value of a variable, repeat itself multiple times, consider whether it would be better to have a small function placed somewhere outside the component. Not everything should be a function of course:
```typescript
const addOne = (numberValue: number): number => {
  return numberValue + 1
}
```
no matter how many times we need to add 1 to something, it would be silly to create a function for this. But something like this:
```typescript
const getProgramId = (programs?: Program[], slug: string): string | null => {
  return programs?.find(p => p.slug === slug)?.id ?? null
}
```
we might need this 3+ times in a single component. So, we make a function, give it a good name and our code becomes more readable
___
- are the variable names clear enough to understand the function of the code?
  - our state variables and function names should be descriptive enough that we can quickly look at some code and at least understand what is its purpose, even if we don't understand the implementation details
  - often times when copying code from some documentation, the code will be purposefully generically named because the example usually is generic. But wi should change this to fit into our implementation:
```typescript
const OurTabs: React.FC = () => {
  const [value, setValue] = useState(0)
  const handleChange = (newValue) => {
    setValue(value)
  }
}
```
looking at this we don't really have a clear idea what exactly `value` is. Small changes make a big difference:
```typescript
const OurTabs: React.FC = () => {
  const [currentTabIndex, setCurrentTabIndex] = useState(0)
  const handleTabClick = (newCurrentTabIndex: number) => {
    setCurrentTabIndex(currentTabIndex)
  }
}
```
---
- is the component structure good
  - do we have too many / too few components? Sometimes there will be a huge component file with code and logic that could be separated into 2+ components (a parent component with children, a wrapper, completely separate components etc.)
  - other times, we can even define too many components. Single-line components are fine as long as we use them in 3+ places. But if we use them in one, we should just leave that in the main component
---
- are there unecessary `console.logs()` or multiple empty lines?
- are there unused files added (for instance, image assets that ended up not being used)
