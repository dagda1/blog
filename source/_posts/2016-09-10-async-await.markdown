---
layout: post
title: "Streams and Async Await in Nodejs"
date: 2016-09-10 12:46:55 +0100
comments: true
categories: Nodejs JavaScript Async
---
Dealing with asynchronicity in nodejs has been a challenge from day one due to its non blocking nature.  The evolution has been slow and the node world has moved from <a href="http://callbackhell.com/" target="_blank">callback hell</a> to <a href="http://www.html5rocks.com/en/tutorials/es6/promises/#toc-parallelism-sequencing" target="_blank">promises</a> and from promises to <a href="http://www.thesoftwaresimpleton.com/blog/2014/03/01/es6-generators/" target="_blank">generators</a>.

Transpilers such as <a href="https://babeljs.io/" target="_blank">babel</a> allow developers to use tomorrow's unratified features of javascript today and <a href="https://h3manth.com/new/blog/2015/es7-features/" target="_blank">ecmascript7's</a> ```async and await``` could prove to be a game changer.

I've used <a href="https://www.google.co.uk/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=async%20await%20c%23" target="_blank">async and await</a> in ```c#``` where its addition is still great but its impact is not quite as dramatic.

With ```async and await```, you can write asynchronous code that for all intensive purposes looks synchronous by marking functions as ```async``` and and prefixing function invocations with ```await``` to indicate execution will be deferred until a promise is returned from the function call after ```await```.

Below is a very simple async function:

{% codeblock async.js %}
async doSomethingAsync () {
  let result;

  try {
    result = await getJSON('/controller/action.json');
  }catch(e) {
    console.dir(e);
  }

  // do something with result
}
{% endcodeblock %}

The function is marked as async with the ```async``` keyword on line 1.  Any function that will ```await``` on the result of a function that returns a promise must be marked as ```async```.

The ```await``` keyword allows you to ```await``` on the result of a promise.  The ```getJSON``` function that is called on line 5 returns a promise and any function that is called with ```await``` must return a promise to ensure execution is returned to the calling code in the event of a resolved promise just as if the function had been called synchronously.  In the event of a promise rejection, the rejection is thrown allowing you to deal with it just like you would with a normal catch handler.  This is as close as javascript has ever become to having normal looking code for non blocking asynchronous function calls.

It is worth pointing out that promises are the glue that makes all this possilbe and they are still one of the most important primitives in node.

##Setup

<a href="https://babeljs.io/" target="_blank">Babel</a> transpiles function calls marked with ```await``` in ```async``` functions to <a href="http://www.thesoftwaresimpleton.com/blog/2014/03/01/es6-generators/" target="_blank">generators</a>.

In order to transpile async and await to javascript generators, you will need to ```npm install``` the <a href="https://www.npmjs.com/package/babel-plugin-transform-async-to-generator" target="_blank">babel-plugin-transform-async-to-generator</a> package and add a reference to the package in your ```.babelrc```.

Below is my ```.babelrc``` that allows me to use async await and other features:
{% codeblock .babelrc %}
{
  "presets": ["stage-3"],
  "plugins": [
    "transform-es2015-modules-commonjs",
    "transform-object-rest-spread",
    "transform-async-to-generator"
  ]
{% endcodeblock %}

## Streams and Async..Await

I recently wrote this <a href="https://github.com/dagda1/knex-csv-transformer/" target="_blank">csv parser</a> that applies transformations from a csv file input to a destination database table using the excellent <a href="http://knexjs.org/" target="_blank">knexjs</a> query builder package.

The package uses <a href="https://github.com/wdavidw/node-csv-parse" target="_blank">csv-parse</a> which transforms the csv file of delimitted rows and columns into arrays or objects.  CSV-Parse implements the node <a href="https://nodejs.org/api/stream.html#stream_class_stream_transform" target="_blank">stream.Transform API</a>.

Csv-parse creates a readable stream that emits ```data``` events each time it encouters a chunk of data.  CSV-Parse allows me to bind to a <a href="https://nodejs.org/api/stream.html#stream_event_readable" target="_blank">readable</a> event that gets passed a row of a csv file for each row in that particular file.

The package I wrote allows the user to specify transformations from a source csv file to a destination table.  Some of these transformations might involve one or more asynchronous actions that could end up as some pretty messy code if I just used promises so ```async``` and await seemed like a great fit.  I will post the original code at the end of the post that used promises and it is very hard to follow and deeply nested.

Below is the code that hooks up the csv-parse stream events to member functions in the class that will handle the events:
{% codeblock hookup.js %}
this.parser.on('readable', this.onReadable);
this.parser.on('end', this.onEnd);
this.parser.on('error', this.onFailed);

this.csv = fs.createReadStream(this.opts.file);
this.csv.pipe( iconv.decodeStream(this.opts.encoding) ).pipe(this.parser);
{% endcodeblock %}

Line 6 specifies that the ```onReadable``` is bound to the ```readable``` event of the stream.

My first attempt at using async and await with the stream is below where I marked the ```onReadable``` function that gets rows of csv data passed to it as an ```async``` function:

{% codeblock asyncOnReadable.js %}
async onReadable() {
  let record = this.parser.read();

  if (record === null) {
    return;
  }

  if (this.parser.count <= 1) {
    this.headers = record;
  } else {
    if(!this.opts.ignoreIf(record)) {
      const record = await this.createObjectFrom(record);
      this.records.push( record );
    }
  }
}
{% endcodeblock %}

The code calls ```createObjectFrom``` on line 12 which is itself an async method that returns a promise and adds it to the ```this.records``` array that will be used to persist the transformed values to the database.

Below is a scaled down version ```createObjectFrom``` which transforms the csv record into a JavaScript hash of values:

{% codeblock createObjectFrom.js %}
async createObjectFrom(record) {
  let obj = {};

  for(let i = 0, l = this.opts.transformers.length; i < l; i++) {
    let transformer = this.opts.transformers[i];

    if(transformer.options.lookUp) {
      const result = await this.knex(lookUp.table).where(whereClause).select(lookUp.scalar);

      if(result.length) {
        csvValue = result[0][lookUp.scalar];
      }
    } else {
      let csvValue = record[i];
    }

    const value = transformer.formatter(csvValue, record, obj);

    if((value != undefined && value != null) && transformer.options.addIf(value)) {
      obj[transformer.field] = value;
    }
  }

  return Promise.resolve(obj);
}
{% endcodeblock %}

The code loops over an array of transformations, one of which might be an async call to ```knex``` on line 9 to retrieve a value from the database.

A promise is returned on line 23 to allow this function to be called with async and await.

I wrote a test to test this function in isolation and I was buoyed when it passed but when running the code for real with stream ```readable``` events being raised, the ```this.records``` array was empty when it the ```end``` event of the stream was reached:

{% codeblock stream.js %}
onEnd() {
    console.dir(this.records); // []
}
{% endcodeblock %}

This is because the ```onReadable``` function is not being called async from the csv-parse module that raises the evnets.

After much head scratching the answer was to return a promise from ```createObjectFrom```.  But the interesting part of this solution was that I was able to mark the function that gets passed to the Promise constructor as ```async``` on line 2 of the code below:

{% codeblock Promise.js %}
createObjectFrom(record) {
  return new Promise(async (resolve, reject) => {
    let obj = {};

    for(let i = 0, l = this.opts.transformers.length; i < l; i++) {
      let transformer = this.opts.transformers[i];

      if(transformer.options.lookUp) {
        const result = await this.knex(lookUp.table).where(whereClause).select(lookUp.scalar);

        if(result.length) {
          csvValue = result[0][lookUp.scalar];
        }
      } else {
        let csvValue = record[i];
      }

      const value = transformer.formatter(csvValue, record, obj);

      if((value != undefined && value != null) && transformer.options.addIf(value)) {
        obj[transformer.field] = value;
      }

    return resolve(obj);
  });
}
{% endcodeblock %}

On line 2, I mark the anonyous function that gets passed to the Promise constructor as ```async``` and this allows me to invoke functions with ```await``` and only resolve the promise on line 24 when all the processing has finished.

The ```onReadable``` event that calls this function now looks like this and is no longer async:

{% codeblock onReadable.js %}
onReadable() {
  let record = this.parser.read();

  if (record === null) {
    return;
  }

  if (this.parser.count <= 1) {
    this.headers = record;
  } else {
    if(!this.opts.ignoreIf(record)) {
      const promise = this.createObjectFrom(record);
      this.promises.push( record );
    }
  }
}
{% endcodeblock %}

```createObjectFrom``` now returns a promise that will resolve later and ```onReadable``` works as advertised.

Lines 12 and 13 simply add each promise that is returned from ```createObjectFrom``` to the ```this.promises``` array that can be processed later.

The ```onEnd``` event now uses ```Promise.all``` to wait for all the promises to resolve before inserting the resolved values into the database:

{% codeblock onEnd.js %}
onEnd() {
  Promise.all(this.promises).then(values => {
    // insert records
  });
}
{% endcodeblock %}

This works nicely and is much better than the Promise handling code at the bottom of this post.

## async and await in tests

Async and await can also be utilised to make your testing code much simpler and synchronous looking.

Below is my test to ensure that the records are being inserted into the database:

{% codeblock test1.js %}
describe('transformer', () => {
  it('transforms the data, imports the csv file and creates the records', async () => {
    const ignoreIf = (data) => data[3] !== 'Liverpool' && data[4] !== 'Liverpool';
    const opts = { table: 'results', file: __dirname + '/fixtures/test.csv', encoding: 'utf8', transformers, ignoreIf: ignoreIf };

    await seeder(opts)(knex, Promise);

    const results = await knex('results');

    expect(results.length).to.equal(2);

    const firstResult = results[0];

    const team_id = await knex('teams').where({name: 'Wimbledon'}).select('id');

    expect(team_id[0].id).to.equal(results[0].team_id);

    expect(manager_id).to.equal(results[0].manager_id);
  });
});
{% endcodeblock %}

Normally when testing promises from mocha, you have to return a promise from the mocha ```it``` function in order for execution to wait until the promise has resolved but as I have marked the anonymous function on line 2 as ```async```, I can simply ```await``` (line 6) for the asynchoronous csv parsing code to finish before testing the results.

I can also make further asynchronous calls in the anonymous function like I am on line 14 and ```await``` their results before asserting expectations.

This is flatter and synchronous looking code that we all know well.

Below is the code I mentioned previously which used promises instead of async await and is pretty damn nasty.

The async await version is considerably cleaner.

Feedback is welcomed in the comments below.

{% codeblock promisehell.js %}
  createObjectFrom(record) {
    const self = this;
    const promises = [];

    return new Promise((resolve, reject) => {
      const getValue = (transformer, csvValue, obj) => {
        const value = transformer.formatter(csvValue, record, obj);

        if((value != undefined && value != null) && transformer.options.addIf(value)) {
          return value;
        }
      };

      for(let i = 0, l = self.opts.transformers.length; i < l; i++) {
        let transformer = self.opts.transformers[i];

        const headerIndex = findIndex(self.headers, (header) => {
          return header === transformer.column;
        });

        let csvValue = record[headerIndex];

        if(transformer.options.lookUp) {
          promises.push(new Promise((resolve, reject) => {
            const lookUp = transformer.options.lookUp;

            const whereClause = {};

            whereClause[lookUp.column] = csvValue;

            self.knex(lookUp.table).where(whereClause).select(lookUp.scalar).then((result) => {
              if(result.length) {
                return resolve({
                  transformer,
                  value: result[0][lookUp.scalar],
                  headerIndex,
                  record
                });
              }else {
                if(lookUp.createIfNotExists && lookUp.createIfNotEqual(csvValue)) {
                  const insert = {[lookUp.column]: csvValue};

                  self.knex(lookUp.table)
                    .insert(insert)
                    .returning('id')
                    .then((inserted) => {
                      return resolve({
                        transformer,
                        value: inserted[0],
                        headerIndex,
                        record
                      });
                    });
                } else {
                  resolve({
                    transformer,
                    value: undefined,
                    headerIndex,
                    record
                  });
                }
              }
            });
          }));
        } else {
          promises.push(Promise.resolve({
            transformer,
            value: csvValue,
            headerIndex,
            record
          }));
        }
      }

      return Promise.all(promises).then((result) => {
        const obj = result.reduce((prev, curr, index, arr) => {
          const value = getValue(curr.transformer, curr.value, prev);

          if(value === undefined && value === null) {
            return prev;
          }

          prev[curr.transformer.field] = value;

          return prev;
        }, {});

        console.dir(obj);

        resolve(obj);
      }).catch((err) => {
        console.log('in Promise.all error');
        console.dir(err);
      });
    });
  }

{% endcodeblock %}
