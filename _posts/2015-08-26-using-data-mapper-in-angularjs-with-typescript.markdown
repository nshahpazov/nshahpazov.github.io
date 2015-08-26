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
  query<T>(params: {}): angular.IPromise<T[]>;
  save<T>(record: any): angular.IPromise<T>;
  create<T>(any): angular.IPromise<T>;
  remove(any): angular.IPromise<any>;
}

{% endhighlight %}

As you can see we define a simple interface following the diagram and describing basic network communication operations done with a model. All operations return a promise which works with a particular domain object **T**ype.

{% highlight js %}
/// <reference path="../../../../typing/object-assign.d.ts" />
import assign = require('object-assign');
import IMapper from '../../common/interfaces/IMapper';
import Client from '../models/Client';

let CLIENT_URI = 'http://awesomestore.com/api/client';

class ClientMapper implements IMapper {
  private httpService: angular.IHttpService;

  public static $inject = ['$http'];

  constructor(private $http: any) {
    this.httpService = $http;
  }

  static construct($http: any): ClientMapper {
    return new ClientMapper($http);
  }

  get(params: {id: number}): angular.IPromise<Client> {
    return this.httpService.get(ClientUris.CLIENT + params.id)
      .then(res => new Client(res.data));
  }

  query(params: {}): angular.IPromise<Client[]> {
    return this.httpService.get(ClientUris.CLIENT)
      .then(R.compose(R.map(Client.construct), get('data'));
  }

  save(client: Client): angular.IPromise<Client[]> {
    let id = client.clientId;
    return this.httpService.put(ClientUris.CLIENT + id, client)
      .then(res => new Client(res.data));
  }

  create(params: {}): angular.IPromise<Client> {
    return this.httpService.post(ClientUris.CLIENT, client)
      .then(res => new Client(res.data));
  }

  remove(params: {}): angular.IPromise<boolean> {
    return $http.delete(MED_URI + this.keyId)
      .then(res => res.status === 200);
  }
}

export default ClientMapper;

{% endhighlight %}