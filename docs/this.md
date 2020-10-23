# Difference of JavaScript `this` keyword in Arrow Function and Regular Fucntion

## Summary
This note document one issue I met with `this` in JavaScript. In previous time, I did not have a in depth understanding about `this`. I hope to talk more about 
`this ` in this docuemnt while I solve the issue I've met. 

## The Problem
I have a block of code from a Vue course. It's a vue instance. Let's focus on the `mounted()` method. This method use axios to get a json data and pass a callback 
function into the `then()`. Inside the callback, we uses a `map()` and pass another callback. We use this in these callbacks point to the vue instance (the object).
This code works as we expected it to be.

```JavaScript
export default {
  name: "MainApp",
  data: function () {
    return {
      appointments: [],
      aptIndex: 0
    };
  },
  components: {
    AppointmentList,
  },
  mounted() {
    axios.get("./data/appointments.json").then(res => (
      this.appointments = res.data.map(item => {
        item.aptId = this.aptIndex;
        this.aptIndex++;
        return item;
      })
    ));
  },
  methods: {
    removeItem: function (apt) {
      //use loadash without method to delete item form an array
      this.appointments = _.without(this.appointments, apt);
    },
  },
};
```


Well, the problem occurs when I change the arrow function to regualr function. The console throw me a error: `Cannot read property 'aptIndex' of undefined`. Of course,
that's becasue `this` is undefined. So my question why `this` in the arrow function doesn't has problem but it doesn't point to the object in the regular function?

This is becuase inside a function, the value of this depends on how the function is called. In the below code, we pass a callback to `map()`, this callback function has their own `this`
, and beucase it is a function, so its `this` are depend on how it is called. In this case, this callback get called without a the value of `this` set , 
`this` will default to the global object, which is window in a browser. 

```JavaScript
  mounted: function mounted() {
    axios.get("./data/appointments.json").then(res => (
      return this.appointments = res.data.map(function (item) {
        item.aptId = this.aptIndex;
        this.aptIndex++;
        return item;
      });
    ));
  }
```
## Arrow Functions
The arrow functions are different. Arrow functions doesn't have its own `this`. In arrow functions, `this` retains the value of the enclosing lexical context's `this`, which means the arrow 
function inherits `this` from its enclosing scope(outter/parent scope). In below code, we change the callback to an arrow function, so its `this` is the context of the enclosing callback,
and the enclosing context inherits the context of `then()`. And the context of `then()` inherits `mounted()`. The `this` of `mounted()` is the vue object becasue `mounted()` is a method.
When a function is called as a method of an object, its `this` is set to the object the method is called on. As a result, use arrow functions will track of  `this`.

```JavaScript
  mounted: function mounted() {
    axios.get("./data/appointments.json").then(res => (
      return this.appointments = res.data.map(item => {
        item.aptId = this.aptIndex;
        this.aptIndex++;
        return item;
      });
    ));
  }
```
## Other Ways to Solve the Issue
If you do not want to use arrow functions. You can store `this` into a varible and use it in the callbacks. The new variable  also refers to the object that `this` refers to.

```Javascript
  mounted: function mounted() {
    var _this = this;

    axios.get("./data/appointments.json").then(function (res) {
      return _this.appointments = res.data.map(function (item) {
        item.aptId = _this.aptIndex;
        _this.aptIndex++;
        return item;
      });
    });
  }
```

References: <br />
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this <br />
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions <br />
https://blog.bitsrc.io/what-is-this-in-javascript-3b03480514a7 <br />
https://stackoverflow.com/questions/43929650/vuejs-why-is-this-undefined <br />
https://stackoverflow.com/questions/20279484/how-to-access-the-correct-this-inside-a-callback <br />
https://vuejs.org/v2/guide/instance.html#Instance-Lifecycle-Hooks <br />
http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html
