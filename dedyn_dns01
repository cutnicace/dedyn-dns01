package main

import (
        "bytes"
        "fmt"
        "github.com/docopt/docopt-go"
        "io/ioutil"
        "net/http"
        "os"
        "strings"
)

var dedynToken = os.Getenv("DEDYN_TOKEN")
var dedynName = os.Getenv("DEDYN_NAME")

const dedynApi = "https://desec.io/api/v1/domains/"

type Exit struct{ Code int }

func handleExit() {
        if e := recover(); e != nil {
                if exit, ok := e.(Exit); ok == true {
                        os.Exit(exit.Code)
                }
                panic(e) // not an Exit, bubble up
        }
}

func sendCommand(method string, ending string, params string){
        var url = dedynApi + ending
        var jsonStr = []byte(params)
        req, err := http.NewRequest(method, url, bytes.NewBuffer(jsonStr))
        req.Header.Set("Authorization", `Token ` + dedynToken)
        req.Header.Set("Accept", "application/json")
        req.Header.Set("Content-Type", "application/json")

        client := http.DefaultClient
        resp, err := client.Do(req)
        if err != nil {
                panic(Exit{1})
        }
        defer resp.Body.Close()

        fmt.Println("response Status:", resp.Status)
        fmt.Println("response Headers:", resp.Header)
        body, _ := ioutil.ReadAll(resp.Body)
        fmt.Println("response Body:", string(body))

        if method == "PUT" && resp.Status != "200 OK" {
                os.Exit(2)
        } else if method == "DELETE" && resp.Status != "204 No Content" {
                os.Exit(3)
        }

}

func addTxtRecord(name string, txt string) {
        var params = `[{"subname":"` + name + `", "type":"TXT", "records":["\"` + txt + `\""], "ttl":60}]`
        var urlend = dedynName + `/rrsets/`
        var action = "PUT"
        sendCommand(action, urlend, params)
}

func delTxtRecord(name string) {
        var params = ""
        var urlend = dedynName + "/rrsets/" + name + "/TXT/"
        var action = "DELETE"
        sendCommand(action, urlend, params)
}

func main() {
        usage := `deSEC.io DNS01 acme exec provider

Usage:
        dedyn_dns present <fqdn> <txt>
        dedyn_dns cleanup <fqdn> <txt>
        dedyn_dns timeout

Options:
        -h --help     Show this screen.
`
        args, _ := docopt.ParseDoc(usage)

        defer handleExit()

        var name = ""

        if args["timeout"] == true {
                //print timeout and polling interval to stdout
                fmt.Println(`{"timeout": 120, "interval": 5}`)
        } else {
                //only fill the variables when given
                var fqdn = strings.Split(args["<fqdn>"].(string), ".")
                //count number of subdomains via dots in dedynName
                var dots = strings.Count(dedynName, ".")
                //add two more for the separator and trailing dot in the fqdn
                dots += 2
                name = strings.Join(fqdn[:len(fqdn)-dots], ".")
        }

        if args["present"] == true {
                addTxtRecord(name, args["<txt>"].(string))
        } else if args["cleanup"] == true {
                delTxtRecord(name)
        }
}
