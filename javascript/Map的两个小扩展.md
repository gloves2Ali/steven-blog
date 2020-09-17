在以前，实现数据映射，我们利用给对象属性赋值，就像这样：
```javascript
var mapping = Object.create( null );
mapping[ key ] = value;
```
ES6 的 Map/WeakMap 提供了更为便利及语义化的用法，功能更加强大，key 支持所有数据类型：
```javascript
const mapping = new Map;
mapping.set( key, value );
```
通过继承 Map/WeakMap，很容易扩展功能，以 Map 为例，本文讲述一个带 过期控制 和 持久化 能力的 Map 扩展。
为表述方便，文中出现的代码均为伪代码，不可以直接运行。

过期控制
基本思路：每个 Map 中的 key，都可以指定其生命周期，在它生命终结后被自动清除。

增加 set 方法的第三个参数，声明该 key 的生命周期，如果超过指定时间，再 get 时拿到的值为 undefined。

伪代码如下：
```javascript
class MemoryStore extends Map{
  constructor( ...args ){
    super( ...args );
  }

  get( key ){
    if( hasExpired( key ) ){
      this.delete( key );
    }

    return super.get( key );
  }

  set( key, value, expires ){
    if( typeof expires === 'number' ){
      setExpires( key, expires );
    }

    return super.set( key, value );
  }
}
```
上面用了 setExpires、hasExpired 两个函数来做生命周期的设定和查询，除了设定和查询外，完整的功能还应该包括删除和清空，而这些方法刚好在原生 Map 中已有实现，所以过期时间的数据索性用一个 Map 来记录。

在 Map 的基础上，扩展 check 方法用于检查一个 key 是否过期：
```javascript
class Expires extends Map {
  check( key ){
    const now = Date.now();
    const expiredTime = this.get( key );

    if( expiredTime ){
      return expiredTime > now;
    }else{
      return true;
    }
  }

  set( key, duration ){
    return super.set( key, Date.now() + duration );
  }
}
```
把 Expires 应用到 MemoryStore 上，作为它附带的一个子对象 expiresControl：
```javascript
class MemoryStore extends Map {
  constructor( ...args ){
    super( ...args );
    this.expiresControl = new Expires;
  }

  get( key ){
    if( !this.expiresControl.check( key ) ){
      this.delete( key );
    }

    return super.get( key );
  }

  set( key, value, expires ){
    if( typeof expires === 'number' ){
      this.expiresControl.set( key, expires );
    }

    return super.set( key, value );
  }

  clear(){
    this.expiresControl.clear();
    return super.clear();
  }

  delete( key ){
    this.expiresControl.delete( key );
    return super.delete( key );
  }
}
```
到此为止，一个带有过期控制的 MemoryStore 就做完了，凡是过期的 key，在获取其 value 的时候总是拿到 undefined。

对 MemoryStore 做进一步扩展，把内存中的数据保存到一个 json 文件中，以供应用重启的时候得到复原，下例我们来实现一个带有这样的持久化能力的 FileStore。

持久化
基本思路：FileStore 除了实现 MemoryStore 的所有能力外，同时需要将数据体和本地的一个 json 文件同步，应用无论如何重启，都不会受到影响。

FileStore 从 MemoryStore 继承而来，并扩展 json 文件同步的能力：
```javascript
const { set, get } = Map.prototype;

class FileStore extends MemoryStore {
  constructor( ...args ){
    super( ...args );
  }

  async load( file ){
    this.bindingFile = file;

    let data = await json.read( file );

    for( let [ key, value, expires ] of data ){
      set.call( this, key, value );

      if( expires ){
        set.call( this.expiresControl, key, expires );
      }
    }
  }

  async save(){
    const content = [];

    for( let [ key, value ] of this ){
      const expires = get.call( this.expiresControl, key );
      const item = [ key, value ];

      if( expires ){
        item.push( expires );
      }

      content.push( item );
    }

    await json.write( this.bindingFile, content );
  }
}
```
load() 方法实现从 json 文件中加载数据，并还原到当前实例中，save() 则相反，将当前实例所携带的数据，保存到 json 文件。需要注意的是，子对象 expiresControl 的数据也需要进行同步。

为了更加自动，可以把 save() 方法应用到对实例数据产生修改的所有方法当中：
```javascript
class FileStore extends MemoryStore {
  ...

  async set( ...args ){
    const result = super.set( ...args );
    await this.save();
    return result;
  }

  async clear( ...args ){
    const result = super.clear( ...args );
    await this.save();
    return result;
  }

  async delete( ...args ){
    const result = super.delete( ...args );
    await this.save();
    return result;
  }

  ...
}
```

这样一来，写数据会同时写到内存和文件中，取数据则只从内存里取。

捆绑销售
为了方便使用，我把 MemoryStore 和 FileStore 打包在一起，用一个函数统一出口，包装了一个叫 MemoryCache 的 npm 模块：
```javascript
const GlobalStore = new Map;

exports.store = async ( name, fileName ) => {
  if( GlobalStore.has( name ) ){
    return GlobalStore.get( name );
  }else{
    let store;

    if( fileName ){
      store = new FileStore;
      await store.load( fileName );
    }else{
      store = new MemoryStore;
    }

    GlobalStore.set( name, store );

    return store;
  }
}
```
store 函数以是否提供 fileName 参数来决定使用 MemoryStore 或者 FileStore，返回一个经过包装的 Map 对象，既可以像原生 Map 一样使用，又支持过期控制和持久化，用法详见 这里。

有什么用？
该模块适合做一些小型的、临时性的用户交互行为信息存贮，举两个小例子：

登录验证码：服务端生成一个随机数，与即将登录的帐号映射，过期时间 n 分钟，随机数以图片形式发给浏览端，要求用户输入，以在后端做校验。
查看数的计算：用户在 n 分钟之间重复查看一篇文章，该文章的查看数只增加 1。可以利用该模块将文章 id 与查看用户做限时映射，如果已存在该用户，则查看数不增加。
