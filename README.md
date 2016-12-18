# qlstore

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

var secret = "EyaC2BPcJtNqU3tjEHy+c+Wmqc1yihYIbUWEl/jk0Ga73kWBclmuSFd9HuJKwJw/Wdsh1XnjY2Bw1HBVph6WOw=="
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
