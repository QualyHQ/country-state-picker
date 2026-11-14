# Country State Picker

Qualy’s checkout of the `country-state-picker` country/state lookup library. It contains static data and pure lookup functions, not a UI picker or an address-validation service. The package metadata retains the upstream author and ISC license.

## Install and use

```bash
npm install country-state-picker
```

That command installs the public upstream package; it does not guarantee the same revision as this Qualy fork. Consumers requiring this checkout’s changes must pin their approved fork revision.

```js
const { getCountries, getCountry, getStates, getFilteredCountries } = require('country-state-picker');

getCountries();
getCountry('au');
getCountry('Australia');
getStates('au');
getFilteredCountries(['au', 'in']);
```

- `getCountries()` returns the backing country array, with `name`, lowercase `code` and `dial_code`. Treat it as read-only: it is not cloned.
- `getCountry(value)` returns the first case-insensitive exact match by name, country code or dial code, or `undefined`. Calling codes are not unique; this is not phone-number parsing.
- `getFilteredCountries(values)` returns matching countries in dataset order, rather than argument order.
- `getStates(codeOrCountry)` accepts a case-insensitive string code or a country object and returns its state-name array, or `[]` for an unknown code. Object codes are used as supplied. The returned array is not cloned.

The bundled [country data](src/data/countries.ts) and [state data](src/data/states.ts) are a static snapshot. Check them before relying on administrative names, boundaries or current dial-code coverage. This library does not certify an address or jurisdiction.

## Development

```bash
npm ci
npm run lint
npm run build
npm test
```

The repository uses TypeScript, TSLint and Mocha. `npm run build` emits `lib/`, which is the package’s declared runtime/type entry point. `npm run format` rewrites TypeScript source; inspect the diff before committing. See [the lookup implementation](src/index.ts) and [package.json](package.json) for the current toolchain.
