# mongoose-class-wrapper [![Build Status][travis-image]][travis-url]
Tiny ES6 class-based wrapper for mongoose model methods.

## Installation

```
$ npm i mongoose mongoose-class-wrapper --save
```

## Usage

You can use this plugin with latest Node.js versions (v4 LTS or v5 Stable) as they support ES6 class syntax (in strict mode). For previous versions use [babel][babel-url].

### Basic Example

```js
import mongoose from 'mongoose';
import loadClass from 'mongoose-class-wrapper';

// Create mongoose schema
var userSchema = new mongoose.Schema({
  username: {type: String, unique: true, required: true},
  email: {type: String, unique: true, required: true},
  hashedPassword: {type: String, required: true},
  salt: {type: String, required: true},
  created: {type: Date, default: Date.now}
});

// Create new class with model methods
class UserModel {

  get password() {
    return this._plainPassword;
  }
  set password(password) {
    this._plainPassword = password;
    this.salt = Math.random() + '';
    this.hashedPassword = this.encryptPassword(password);
  }

  encryptPassword(password) {
    return crypto.createHmac('sha1', this.salt).update(password).digest('hex');
  }

  static byEmail(email) {
    return this.findOne({email}).exec();
  }

}

// Add methods from class to schema
userSchema.plugin(loadClass, UserModel);

// Export mongoose model
export default mongoose.model('User', userSchema);
```

### Inheritance Example

```js
import mongoose from 'mongoose';
import loadClass from 'mongoose-class-wrapper';

class Resource {}

// Add schema to base class
Resource.schema = {
  version: {
    type: Number,
    default: 0
  }
}

// Add hooks to base class
Resource.hooks = {
  pre: {
    save: function(next) {
      this.version += 1;
      next();
    }
  }
}

var schema = new mongoose.Schema({
  title: String,
  author: String
});

class Book extends Resource {}

// Add schema, hooks and methods from class to schema
schema.plugin(loadClass, Book);

// Export mongoose model
export default mongoose.model('Book', schema);

```

## ES7 decorators

ES7 decorators for mongoose models are available in [mongoose-decorators][mongoose-decorators-url] package.

## License
This library is under the [MIT License][mit-url]


[travis-image]: https://img.shields.io/travis/aksyonov/mongoose-class-wrapper/master.svg
[travis-url]: https://travis-ci.org/aksyonov/mongoose-class-wrapper
[babel-url]: http://babeljs.io/
[decorators-url]: https://github.com/wycats/javascript-decorators
[mongoose-decorators-url]: https://github.com/aksyonov/mongoose-decorators
[mit-url]: http://opensource.org/licenses/MIT
