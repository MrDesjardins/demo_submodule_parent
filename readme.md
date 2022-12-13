This [repository](https://github.com/MrDesjardins/demo_submodule_parent) uses a submodule Git [repository](https://github.com/MrDesjardins/demo_submodule_child).

# Steps to Create the SubModule Relationship

The first step is to add the submodule to this project.

```sh
git submodule add https://github.com/MrDesjardins/demo_submodule_child ./packages/demo_submodule_child
```

The second step is to modify the `tsconfig.json` to have an alias. It is not _required_ but make the code from the `src` folder of the parent repository to access any submodule package easily.

```json
"paths": {
  "@demo_submodule_child": ["packages/demo_submodule_child/src"],
  "@demo_submodule_child/*": ["packages/demo_submodule_child/src/*"]
}
```

The third step is to add two registries to Node. The following command compiles Typescript into JavaScript and then it calls Node with the two registrys following with the JavaScript entry file.

```json
"start": "npx tsc && node -r ts-node/register/transpile-only -r tsconfig-paths/register build/src/index.js"
```

# How to Modify the SubModule Code

There are several patterns. You can have in a complete other folder the repository and do a change, then push the change. Then come back to the consumer folder and pull the changes. However, that is a lot of steps.

A more straightforward pattern is to modify the Submodule directly into the consuming (parent) project.
