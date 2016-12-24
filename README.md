# qlstore

[![GoDoc](https://godoc.org/github.com/gernest/qlstore?status.svg)](https://godoc.org/github.com/gernest/qlstore) [![Build Status](https://travis-ci.org/gernest/qlstore.svg?branch=master)](https://travis-ci.org/gernest/qlstore) [![Coverage Status](https://coveralls.io/repos/github/gernest/qlstore/badge.svg?branch=master)](https://coveralls.io/github/gernest/qlstore?branch=master) [![Go Report Card](https://goreportcard.com/badge/github.com/gernest/qlstore)](https://goreportcard.com/report/github.com/gernest/qlstore)

Implements [gorilla/sessions](https://github.com/gorilla/sessions) store using embedded sql database ( [ql](https://github.com/cznic/ql))

# installation

	go get github.com/gernest/qlstore

# Usage
```go

package main

import (
	"database/sql"
	"log"
	"net/http"

	"github.com/gernest/qlstore"
	// load ql drier
	_ "github.com/cznic/ql/driver"
)

var keyPair = [][]byte{
	[]byte("ePAPW9vJv7gHoftvQTyNj5VkWB52mlza"),
	[]byte("N8SmpJ00aSpepNrKoyYxmAJhwVuKEWZD"),
}

func main() {

	db, err := sql.Open("ql-mem", "testing.db")
	if err != nil {
		log.Fatal(err)
	}

	// This is a convenient helper. It creates the session table if the table
	// doesnt exist yet.
	err = qlstore.Migrate(db)
	if err != nil {
		log.Fatal(err)
	}

	store := qlstore.NewQLStore(db, "/", 2592000, keyPair...)
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// Get a session. We're ignoring the error resulted from decoding an
		// existing session: Get() always returns a session, even if empty.
		session, _ := store.Get(r, "session-name")
		// Set some session values.
		session.Values["foo"] = "bar"
		session.Values[42] = 43
		// Save it before we write to the response/return from the handler.
		session.Save(r, w)
	})
	log.Fatal(http.ListenAndServe(":8090", nil))
}
```
