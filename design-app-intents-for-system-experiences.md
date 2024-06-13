# Design App Intents for system experiences

Presenter: Ray Pai, Apple Design Team

Link: https://developer.apple.com/wwdc24/10176

App Intents surface functionality outside of your app
- Siri, widgets, shortcuts, actions, etc.

What does an app intent look like?
- Summary containing the App, a verb, and parameters that people need to fill out to run the intent

Updated guidance about how to design app intents

## Which App Intents to make?

In iOS 18, anything your app does should be an app intent - it's no longer focused on habitual stuff

Intents that start with these verbs are fundamental things you should provide:

- Get
- Edit
- Create
- Delete
- Open

Avoid making several different intents for the same task. Structure your app's functionality into a flexible intent with parameter.
App intents should not trigger specific UI elements, but instead be focused on the task

## How to structure

- Parameter summaries
- If it needs input, choose from the Apple library
- If it needs options not covered by the basics, use a static parameter for those options
- For options that may change over type create a dynamic entity
- Use optional parameters to allow a user to take action immediately even if they do not supply the parameter
- Toggles: if an intent only changes between two states, support toggle as a default parameter
- Open intents: opening your app is a common behavior, so don't try to avoid doing it.
  - OpenIntent inherently functions to open your app
  - Open the app to show action in app (change, such as creating a new thing)
