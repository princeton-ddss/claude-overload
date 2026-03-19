# Plugins

1. Start a new session
2. Install `ralph-loop` plugin with `/plugin`
3. Show the plugin is listed as installed (run `/reload-plugins`, if needed)
4. Show the plugin is added to `.claude/settings.json`
5. Use the plugin to build a predictive model (push to background?):

```shell
/ralph-loop:ralph-loop "Build a predictive model of county-level election Republican vote share using ACS variables. Requirements: train-validate-test split. Hold-out R-squared above 75%. Only use linear regression methods. Output <promise>COMPLETE<promise> when done." --completion-promise "COMPLETE" --max-iterations 5
```
