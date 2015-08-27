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

<style>
  .btn {
    border-radius: 12px;
    background-color: #336699;
    cursor: pointer;
  }
</style>

<xscript src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js" charset="utf-8"></xscript>

<script></script>

In this post I'm going to show you a way of separating your concerns when it comes to dealing with the network communication of your application. First, lets start with a simple diagram.
<p><br/></p>


<img src="http://eisuke.github.io/yapcasia2010/datamapper.png"
  alt="Data Mapper Diagram" width="960" height="410">

<p><br /></p>
<p><br /></p>

Data Mapper is an architectual pattern for separating the business logic of your domain model from the network communication logic.

<!-- quote -->
"A Data Mapper is a Data Access Layer that performs bidirectional transfer of data between a persistent data store (often a relational database) and an in memory data representation (the domain layer).
"
<p><br /></p>

<br />
OK, but since we are interested in the practical use, lets start with the contract.


{% highlight python %}
interface IMapper {
  get<T>(params: {}): angular.IPromise<T>;
  query<T>(): angular.IPromise<T[]>;
  save<T>(record: any): angular.IPromise<T>;
  create<T>(): angular.IPromise<T>;
  remove(record: any): angular.IPromise<any>;
}

{% endhighlight %}

As you can see we define a simple interface following the diagram and describing basic network communication operations done with a model. All operations return a promise which works with a particular domain object **T**ype.

{% highlight js %}
/// <reference path="../typings/object-assign.d.ts" />
/// <reference path="../typings/ramda.d.ts" />
import * as R from 'ramda';
import assign = require('object-assign');
import IMapper from '../interfaces/IMapper';
import User from '../models/User';

let USER_URI = 'http://gotinhost.com/api/users/';

class UserMapper implements IMapper {
  private httpService: angular.IHttpService;

  public static $inject = ['$http'];

  constructor(private $http: any) {
    this.httpService = $http;
  }

  static construct($http: any): UserMapper {
    return new UserMapper($http);
  }

  get(params: {id: number}): angular.IPromise<User> {
    return this.httpService.get(USER_URI + params.id)
      .then(res => new User(res.data));
  }

  query(): angular.IPromise<User[]> {
    return this.httpService.get(USER_URI)
      .then(R.compose(R.map(User.construct), get('data')));
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

