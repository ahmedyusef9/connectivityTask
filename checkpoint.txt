package main

import (
	"encoding/json"
	"log"
	"math/rand"
	"net/http"
	"sync"
	"time"
)

const SUCCESSFUL = "successful"
const FAILED = "failed"

type Address struct {
	IP    string
	Port uint16
}
type Result struct {
	Successful []Address `json:"successful"`
	Failed []Address `json:"failed"`
}
var (
	bufferV,bufferX  = []Address{}, []Address{}
	m = &sync.Mutex{}
	hosts = []Address{
		Address{
			IP: "192.168.2.1",
			Port: 8080,
		},
		Address{
			IP: "192.168.2.2",
			Port: 80,
		},
		Address{
			IP: "192.168.2.3",
			Port: 9000,
		},
		Address{
			IP: "127.0.0.1",
			Port: 8080,
		},
		Address{
			IP: "127.0.0.2",
			Port: 8082,
		},
		Address{
			IP: "127.0.0.3",
			Port: 8083,
		},
		Address{
			IP: "127.0.0.4",
			Port: 8084,
		},
	}
)

func TestConnectivity(host Address) string {
	log.Println("START POLLING: ",time.Now(), host)
	//url := fmt.Sprintf("http://%s:%d", host.IP, host.Port)
	//_, err := http.Head(url)
	random := rand.Intn(60)
	time.Sleep(time.Duration(random) * time.Second)
	log.Println("FINISH POLLING: ",time.Now())
	if 0 == (random % 2) {
		//log.Println("Error", host.IP, err)
		return FAILED
	}
	return SUCCESSFUL
}

func OnNewAddressesForConnectivityTest(hosts []Address) {
	wg := &sync.WaitGroup{}
	wg.Add(len(hosts))
	for _, host := range hosts{
		go func(host Address, wg *sync.WaitGroup) {
			switch TestConnectivity(host) {
			case SUCCESSFUL:
				m.Lock()
				bufferV = append(bufferV, host)
				m.Unlock()
			case FAILED:
				m.Lock()
				bufferX = append(bufferX, host)
				m.Unlock()
			}
			wg.Done()
		}(host, wg)
	}
	wg.Wait()
}

// log the hosts
func GetConnectivityInfo(){
	m.Lock()
	log.Println("***** GetConnectivityInfo *****")
	log.Println(SUCCESSFUL, bufferV)
	bufferV  = bufferV[:0]
	log.Println(FAILED, bufferX)
	bufferX = bufferX[:0]
	m.Unlock()
}

// to handle server requests
func handleAddress(w http.ResponseWriter, req *http.Request) {
	log.Println(req.Body)
	switch req.Method {
	case "GET":
		res := Result{bufferV, bufferX}
		m.Lock()
		bufferV = bufferV[:0]
		bufferX = bufferX[:0]
		m.Unlock()
		b, error := json.Marshal(res)

		if error != nil {
			log.Println(error)
			w.Write([]byte("ERROR"))
		}
		w.Header().Set("Content-Type", "application/json")
		w.Write(b)
	case "POST":
		res := []Address{}
		json.NewDecoder(req.Body).Decode(&res)
		go OnNewAddressesForConnectivityTest(hosts)
		w.Write([]byte("POST REQUEST"))
	}
}
func main() {
	// call goroutine function by "go"
	go OnNewAddressesForConnectivityTest(hosts)
	go OnNewAddressesForConnectivityTest(hosts)
	go OnNewAddressesForConnectivityTest(hosts)
	// Get info look at terminal to see the arrays
	GetConnectivityInfo()
	time.Sleep( time.Duration(rand.Intn(15)) * time.Second)
	GetConnectivityInfo()
	go OnNewAddressesForConnectivityTest(hosts)
	time.Sleep( time.Duration(rand.Intn(15)) * time.Second)
	GetConnectivityInfo()
	go OnNewAddressesForConnectivityTest(hosts)
	defer GetConnectivityInfo()
	time.Sleep( time.Duration(rand.Intn(15)) * time.Second)

	log.Println("Server start on port 5000")
	http.HandleFunc("/address", handleAddress)
	http.ListenAndServe(":5000", nil)
}
