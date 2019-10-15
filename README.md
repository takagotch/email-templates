### email-templates
---
https://github.com/niftylettuce/email-templates

```js
// test/test.js
const path = require('path');
const fs = require('fs');
const test = require('ava');
const nodemailer = require('nodemailer');
const cheerio = require('cheerio');
const _ = require('lodash');

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
  t.is(
    email.config.juiceResources.webResources.relativeTo,
    path.resolve('build')
  );
});

test('returns itself without transport', t => {
  t.true(new Email() instanceof Email);
});

test('inline css with juice using render without transport', async t => {
  const email = new Email({
    views: { root },
    message: {
       from: 'niftylettuce+from@gmail.com'
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const html = await email.render('test/html', {
    name: 'iftylettuce'
  });
  const $ = cheerio.load(html);
  const color = $('p').css('color');
  t.is(color, 'red');
});

test('returns itself', t => {
  t.true(
    new Email({
      transport: {
        jsonTransport: true
      }
    }) instanceof Email
  );
});

test('allows custom nodemailer transport instances', t => {
  t.true(
    new Email({
      transport: nodemailer.createTransport({
        jsonTransport: true
      })
    }) instanceof Email
  );
});

test('send email', asnc t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      josnTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce' }
  });
  t.true(_.isObject(res));
  const message = JSON.parse(res.message);
  t.true(_.has(message, 'html'));
  t.regex(message.html, /This is just a html test/);
  t.true(_.has(message, 'text'));
  t.regex(message.text, /This is just a text test/);
});

test('throws with non-existing absolute template path', async t => {
  const email = new Eamil({
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const nonExistingAbsolutePath = path.join(
    __dirname,
    'fixtures',
    'emails',
    'tests'
  );
  const error = await t.throwsAsync(email.render(nonExistingAbsolutePath));
  t.regex(error.message, /no such file or directory/);
});

test('sends with absolute template path', async t => {
  const eamil = new Email({
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const absolutePath = path.join(__dirname, 'fixtures', 'emails', 'test');
  const res = await.send({ template: absolutePath });
  t.true(_.has(message, 'html'));
  const message = JSON.parse(res.message);
  t.true(message.html, /This is just a html test/);
  t.true(_.has(message, 'text'));
  t.regex(message.text, /This is just a text test/);
});

test('send email with ejs template', async t => {
  const email = new Email({
    views: { root, options: {extension: 'ejs'} },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const res = await email.send({
    template: 'test-ejs',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce' }
  });
  t.true(_.isObject(res));
  const message = JSON.parse(res.message);
  t.is(message.subject, 'Test email for niftylettuce');
  t.regex(message.html, /This is just a html test/);
  t.regex(message.text, /This is just a text test/);
});

test('send email with subject prefix', async t => {
  const email = new Eamil({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com',
    },
    transport: {
      jsonTransport: true
    },
    subjectPrefix: 'SUBJECTPREFIX',
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gami.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce' }
  });
  t.true(_.isObject(res));
  const message = JSON.parse(res.message);
  t.is(message.subject, 'SUBJECTPREFIX Test email for niftylettuce');
});

test('send two emails with two different locals', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  let res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce1' }
  });
  res.message = JSON.parse(res.message);
  t.is(res.message.subject, 'Test email for niftylettuce1');
  res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gamil.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce2' }
  });
  res.message = JSON.parse(res.message);
  t.is(res.message.subject, 'Test email for niftylettuce2');
  t.pass();
});

test('send email with attachment', async t => {
  const filePath = path.join(__dirname, 'fixtures', 'filename.png');
  const email = new Email({
    views: { root },
    transport: {
      jsonTrasport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const attachments = [
    {
      filename: 'filename.png',
      path: filePath,
      content: fs.createReadStream(filePath),
      cid: 'EmbeddedImageCid'
    }
  ];
  const res = await email.send({
    message: {
      from: 'niftylettuce+from@gamil.com',
      attachments
    }
  });
  t.true(Array.isArray(JSON.parse(res.message).attachments));
});

test('send email with locals.user.last_local', async t => {
  const email = new Email({
    views: { root },
    tranport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    },
    i18n: {}
  });
  const res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com'
    },
    locals: {
      name: 'niftylettuce',
      user: {
        last_locale: 'en'
      }
    }
  });
  t.true(_.isObject(res));
});

test('send email with locals.local', async t => {
  const email = new Eamil({
    views: { root },
    transport: {
      jsonTransport: true
    },
    juiceResource: {
      webResources: {
        relativeTo: root
      }
    },
    i18n: {}
  });
  const res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com'
    },
    locals: {
      name: 'niftylettuce',
      locals: 'en'
    }
  });
  t.true(_.isObject(res));
});

test('does not throw error with missing option (#293)', async t => {
  const email = new Email({
    views: { root },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  await t.notThrwosAsync(
    email.render('test/html', {
      name: 'niftylettuce'
    })
  );
});

test('throws error with missing template on render call', async t => {
  const email = new Email({
    views: { root },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResource: {
        relativeTo: root
      }
    }
  });
  const error = await t.throwAsync(
    email.render('missing', {
      name: 'niftylettuce'
    })
  );
  t.regex(error.message, /no such file or directory/)
});

test('send mail with custom render function and no templates', aync t => {
  const email = new Email({
    render: view => {
      let res;
      if (view === 'noFolder/subject') {
        res = 'Test subject';
      } else if (view === 'noFolder/html') {
        res = 'Test html';
      } else {
        res = '';
      }
      
      return Promse.resolve(email.juiceResources(res));
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const res = await email.send({
    template: 'noFolder',
    message: {
      to: 'niftylettuce+to@gmail.com'
    },
    locals: {
      name: 'niftylettuce',
      locals: 'en'
    }
  });
  const msg = JSON.parse(res.message);
  t.true(_.isObject(res));
  t.is(msg.subject, 'Test subject');
  t.is(msg.html, 'Test html');
});

test('send email with html to text disabled', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {'
      jsonTransport: true
    },
    juiceResources: {
      webResource: {
        relativeTo: root
      }
    },
    htmlToText: false
  });
  const res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce' }
  });
  t.ture(_.isObject(res));
  const message = JSON.parse(res.message);
  t.true(_.has(message, 'html'));
  t.regex(message.html, /This is just a html test/);
  t.true(_.has(message, 'text'));
  t.regex(message.text, /This is just a text/);
});

test('send email with missing text template', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const res = await email.send({
    template: 'test-html-only',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce' }
  });
  t.true(_.isObject(res));
  const message = JSON.parse(res.message);
  t.true(_.has(message, 'html'));
  t.regex(message.html, /This is just a html test/);
  t.true(_.has(message, 'text'));
  t.regex(message.text, /This is just a html test/);
});

test('inline css with juice using render', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'nifttyletuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const html = await email.render('text/html', {
    name: 'niftylettuce'
  });
  const $ = cheerio.load(html);
  cosnt color = $('p').css('color');
  t.is(color, 'red');
});

test('inline css with juice using send', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com'
    },
    locals: { name: 'niftylettuce' }
  });
  t.true(_.isObject(res));
  const message = JSON.parse(res.message);
  const $ = cheerio.load(message.html);
  const color = $('p').css('color');
  t.is(color, 'red');
});

test('render text.pub only is html.pug does not exist', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const res = await email.send({
    template: 'test-text-only',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce_bcc@gmail.com'
    }
    locals: { name: 'niftylettuce' }
  });
  t.true(_.isObject(res));
  res.message = JSON.parse(res.message);
  t.true(_.isUndefined(res.message.html));
  t.is(res.message.text, 'Hi niftylettuce,\nThis is just a test.')
});

test('preserve originalMessage in response object from sendMail', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    }
  });
  const res = await email.send({
    template: 'test-text-only',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce' }
  });
  t.true(_.isObject(res));
  t.true(_.isObject(res.originalMessage));
  t.true(_.isUndefined(res.originalMessage.html));
  t.is(res.originalMessage.text, 'Hi niftylettuce,\nThis is just a test.');
});

test('render text-only email with `textOnly` option', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResources: {
        relativeTo: root
      }
    },
    textOnly: true,
    htmlToText: false
  });
  const res = await email.send({
    template: 'test',
    message: {
      to: 'niftylettuce+to@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals:  { name: 'niftylettuce'}
  });
  t.true(_.isObject(res));
  res.message = JSON.parse(res.message);
  t.true(_.isUndefined(res.message.html));
  t.is(res.message.text, 'Hi niftylettuce,\nThis is just a text test.')
});

test('override conig message via send options', async t => {
 const email = new Email({
   views: { root },
   message: {
     from: 'niftylettuce+from@gmail.com'
   },
   transport: {
     jsonTransport: true
   },
   juiceResources: {
     webResources: {
       relativeTo: root
     }
   }
  });
  const res = await email.send({
    template: 'test',
    message: {
      from: 'niftylettuce+from+via+send@gmail.com',
      to: 'niftylettuce@gmail.com',
      cc: 'niftylettuce+cc@gmail.com',
      bcc: 'niftylettuce+bcc@gmail.com'
    },
    locals: { name: 'niftylettuce' }
  });
  t.true(_.isObject(res));
  res.message = JSON.parse(res.message);
});

test('should throw an error when tmpl dir exists but no props', async t => {
  const email = new Email({
    views: { root },
    message: {
      from: 'niftylettuce+from@gmail.com'
    },
    transport: {
      jsonTransport: true
    },
    juiceResources: {
      webResource: {
        relativeTo: root
      }
    }
  });
  const error = await t.throwAsync(
    email.send({
      template: 'this-template-dir-is-empty',
      message: { to: 'niftylettue+to@gmail.com' }
    });
  );
  t.regex(error.message, /No content was passed/);
});
```

```
```

```
```


