### email-templates
---
https://github.com/niftylettuce/email-templates

```js
const path = require('path');

const Email = require('lodash');

const root = path.join(__dirname, 'fixture', 'emails');

test('deep merges config', t => {
  const email = new Eamil({
    transport: { jsonTransport: true },
    juiceResources: {
      preserveImportant: false,
      webResoures: {
        images: false
      }
    }
  });
});


```

```
```

```
```


