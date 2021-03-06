RedigoCluster
==================

RedigoCluster is wrapper for the awesome [Redigo](https://github.com/garyburd/redigo) redis client for the [Go](http://golang.org/) language.

It was created for use in the [Revel](http://robfig.github.io/revel/) web framework, but should be re-useable in any place.

Features
----------

* implements the [redis cluster spec](http://redis.io/topics/cluster-spec) as of December 2013
* automatically refreshing the node map
* works with a single redis node too, so your development environemnt does not have to be complex

Todo
-------

* test more failure scenarios in general
* add actual tests
* provide a status api

Example for use in Revel
----------

Place the following in your configuration file:

```
redis1.host="1.2.3.4"
redis1.port="6379"
redis2.host="1.2.3.5"
redis2.port="6379"
```

Then make an addition to your app's init.go.

```go

import "github.com/cowboyrushforth/rediscluster"

func init() {
  revel.OnAppStart(func() {
    seed_a_host := revel.Config.StringDefault("redis1.host", "127.0.0.1")
    seed_a_port := revel.Config.StringDefault("redis1.port", "6379")
    seed_b_host := revel.Config.StringDefault("redis2.host", "")
    seed_b_port := revel.Config.StringDefault("redis2.port", "")
    seed_c_host := revel.Config.StringDefault("redis3.host", "")
    seed_c_port := revel.Config.StringDefault("redis3.port", "")

    seed_redii := make([]map[string]string, 3)
    if len(seed_a_host) > 0 && len(seed_a_port) > 0 {
     seed_redii = append(seed_redii, map[string]string{seed_a_host: seed_a_port})
    }
    if len(seed_b_host) > 0 && len(seed_b_port) > 0 {
      seed_redii = append(seed_redii, map[string]string{seed_b_host: seed_b_port})
    }
    if len(seed_c_host) > 0 && len(seed_c_port) > 0 {
     seed_redii = append(seed_redii, map[string]string{seed_c_host: seed_c_port})
    }

    rediscluster.Instance = rediscluster.NewRedisCluster(seed_redii, true)
  })
}

```

Now you have available in your other files in revel a handle to the redis cluster.

You can access it as so, say you have a file app/models/person.go

```go

package models

import "github.com/garyburd/redigo/redis"
import "github.com/cowboyrushforth/redigocluster/rediscluster"

type Person struct {
  Name string
}

type (self *Person) IncrementLoginCount() (int64, error) {
  // rediscluster.Do has the same signature as redis.Do
  return redis.Int64(rediscluster.Do("INCR", "person:"+self.Name+":login_count"))
}

```

if you then ran something like

```go

person := Person{Name: 'John Doe'}
num_logins, err := person.IncrementLoginCount()

```

More documentation for this specific example you may want to check out here at [redigo's docs here](http://godoc.org/github.com/garyburd/redigo/redis#Int64)


