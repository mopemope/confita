# confita

confita is a tool that loads configuration from multiple backends and stores it in a struct.

## Install

```sh
go get -u github.com/heetch/confita
```

## Usage

```go
package main

import (
  "log"
  "time"

  "github.com/coreos/etcd/clientv3"
  "github.com/heetch/confita"
  "github.com/heetch/confita/etcd"
)

type Config struct {
  Host        string `config:"host"`
  Port        int    `config:"port"`
  Interval    uint64 `config:"interval"`
  DatabaseURI string `config:"databaseUri"`
}

func main() {
  var cfg Config

  // By default, the loader loads keys from the environment.
  loader := confita.NewLoader()
  err := loader.Load(&cfg)
  if err != nil {
    log.Fatal(err)
  }

  // From the environment and etcd, with a timeout of 5 seconds.
  client, err := clientv3.New(clientv3.Config{
    Endpoints: endpoints,
  })
  if err != nil {
    log.Fatal(err)
  }
  defer client.Close()

  loader = confita.NewLoader(
    confita.Backends(
      confita.EnvBackend(),
      etcd.NewBackend(client, "prefix"),
    ),
    confita.Timeout(5*time.Second),
  )
  err = loader.Load(&cfg)
  if err != nil {
    log.Fatal(err)
  }
}
```