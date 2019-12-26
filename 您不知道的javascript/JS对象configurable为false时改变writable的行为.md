JavaScrpt高级程序设计第三版中，说对象属性的configurable特性为false时，Object.defineProperty不可再修改除writable之外的特性，经过试验，实际情况并非如此，此时， **若writable原来为true，仍然可以改为false；但是如果writable原为false，则不可再修改为true**

``` javascript
var person = {
    name: '实际名字',
    age: '24'
};
Object.defineProperty(person, 'name', {
    configurable: false,
    writable: true,
});
person.name = '虚假名字1';
console.log(person.name) //虚假名字1
Object.defineProperty(person, 'name', {
    writable: false,
});
person.name = '虚假名字2';
console.log(person.name) //虚假名字1
```
**以上configurable为false时，writable由true修改为false成功**

``` javascript
var person={name:'实际名字',age:'24'};
Object.defineProperty(person,'name',{
  configurable:false,
  writable:false,
});
person.name='虚假名字1';         //实际名字
console.log(person.name)
Object.defineProperty(person,'name',{
  writable:true,
});
person.name='虚假名字2';
console.log(person.name)       //TypeError: Cannot redefine property: name
```

以上，**configurable为flase时，writable由false修改为true报错**




