---
layout: post
title: Using Data Mapper in AngularJS with Typescript
author: Nikola Shahpazov
modified:
excerpt:
tags: []
image:
  feature:
date: 2015-08-26T23:16:26+03:00
---

<style></style>
<script></script>

In this post I'm going to show you a way of separating your concerns when it comes to dealing with the network communication of your application. First, lets start with a simple diagram.
<p><br/></p>


<img src="http://eisuke.github.io/yapcasia2010/datamapper.png"
  alt="Data Mapper Diagram" width="960" height="410">

<p><br /></p>
<p><br /></p>

Data Mapper is an architectural pattern for separating the business logic of your domain model from the network communication logic.

<!-- quote -->
>A Data Mapper is a Data Access Layer that performs bidirectional transfer of data between a persistent data store (often a relational database) and an in memory data representation (the domain layer).

You can read more about it <a href="http://martinfowler.com/eaaCatalog/dataMapper.html">here.</a>

<p><br /></p>

<br />
OK, but since we are interested in the practical use, lets start with the contract.


{% highlight python %}
interface IMapper {
  get<T>(id: number): angular.IPromise<T>;
  query<T>(): angular.IPromise<T[]>;
  save<T>(record: any): angular.IPromise<T>;
  create<T>(): angular.IPromise<T>;
  remove(record: any): angular.IPromise<any>;
}

{% endhighlight %}

We define a simple interface following the diagram and describing basic network communication operations done with a model. All operations return a promise which works with a particular domain object **T**ype.

{% highlight js %}
/// <reference path="../typings/object-assign.d.ts" />
/// <reference path="../typings/ramda.d.ts" />
import * as R from 'ramda';
import assign = require('object-assign');
import IMapper from '../interfaces/IMapper';
import User from '../models/User';

let USER_URI = 'http://gotinhost.com/api/users/';

let get = R.curry((prop, obj) => obj[prop]);

class UserMapper implements IMapper {
  private httpService: angular.IHttpService;

  public static $inject = ['$http'];

  constructor(private $http: any) {
    this.httpService = $http;
  }

  static construct($http: any): UserMapper {
    return new UserMapper($http);
  }

  get(id: number): angular.IPromise<User> {
    return this.httpService.get(USER_URI + id)
      .then(res => new User(res.data));
  }

  query(): angular.IPromise<User[]> {
    return this.httpService.get(USER_URI)
      .then(R.compose(R.map(data => new Client(data)), get('data')));
  }

  save(user: User): angular.IPromise<User> {
    let id = user.userId;
    return this.httpService.put(USER_URI + id, user)
      .then(res => new User(res.data));
  }

  create(): angular.IPromise<User> {
    return this.httpService.post(USER_URI, user)
      .then(res => new User(res.data));
  }

  remove(user: User): angular.IPromise<boolean> {
    return $http.delete(USER_URI + user.id)
      .then(res => res.status === 200);
  }
}

{% endhighlight %}

As you can see we are just implementing the interface according to our domain object needs. We are using ramda functional programming library for expressing ourselves in a more declarative way. An example domain object might be something like this:

{% highlight js %}
class User {

  private id: number;
  private prop1: string;
  private prop2: string;
  private prop3: string;

  constructor(data: {}) {
    assign(this, data);
  }

  purchase(item: IShopItem) {
    // purchase business logic
  }
  sell(item: IShopItem) {
    // sell business logic
  }
}
{% endhighlight %}

The **construct** method is just a wrapper for the constructor of the class so that we don't have to use the **new** keyword all the time and we can pass it in other functions arguments. We declare our classes as angular dependencies in the following manner.

{% highlight js %}
angular.module('app')
  .service('ClientMapper', ClientMapper.construct)
  .factory('Client', Client);
{% endhighlight %}

###Conclusion

Using the UserMapper and the User class, we have a very clear separation of concerns. The User domain object doesn't know anything about the network communication, all it cares about is it's own business logic.