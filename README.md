# Ejected Create React App require.context Workaround for Jest

## Overview

Create-React-App instantiated from [create-react-app](https://create-react-app.dev/) and [ejected](https://create-react-app.dev/docs/available-scripts/#npm-run-eject)) for the purposes of advanced testing configuration.

Namely, `require.context` which is provided as a [webpack function](https://webpack.js.org/guides/dependency-management/#requirecontext) is not recognized by jest and thus requires a workaround. As this is a common issue there are several Babel plugins available to resolve this issue (see [below](#alternatives)). The goal of this exercise was to provide a quick blueprint for how to set up this plugin from extant ejected CRA configuration.

## Steps

I elected to go with [babel-plugin-require-context-hook](https://www.npmjs.com/package/babel-plugin-require-context-hook) which required these additional setup steps:

__Install package__
```
npm i babel-plugin-require-context-hook
```

__Modify `config\jest\babelTransform.js`__
```javascript
module.exports = babelJest.createTransformer({
    ...
    babelrc: true,
    ...
});
```

__Create `.babelrc`__
```
{
    "env": {
      "test": {
        "plugins": ["require-context-hook"]
      }
    }
}
```

In __package.json__, remove any extant babel config as duplicate (copy values to `.babelrc` to consolidate into one config)

In `src/setupTests.js`, add registration instantiation:

```javascript
import registerRequireContextHook from 'babel-plugin-require-context-hook/register';
registerRequireContextHook();
```

### Alternatives

[babel-plugin-transform-require-context](https://github.com/asapach/babel-plugin-transform-require-context)

- Creates a dummy reference to `require.context` to replace when functionality is not needed

[require-context.macro](https://www.npmjs.com/package/require-context.macro)

- Utilizes Babel macros (a different flavor of global overrides?), also branched off of require-context-hook
- For use with Storybook setups; but appears to essentially swap out `require.context` call for custom global `requireContext` call that checks availability of webpack function before providing a backup/mock(?) call

[Alternatives to Ejecting](https://create-react-app.dev/docs/alternatives-to-ejecting/)

- Alternatively, create-react-app encourages forking from `react-scripts` to custom-add scripts without the need to maintain all aspects of configuration long-term
