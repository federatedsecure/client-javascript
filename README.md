![license](https://img.shields.io/github/license/federatedsecure/client-javascript)
![CodeQL](https://github.com/federatedsecure/client-javascript/workflows/CodeQL/badge.svg)

# Usage

Either source the script directly from GitHub by

```html
<script src="https://cdn.jsdelivr.net/gh/federatedsecure/client-javascript/src/federatedsecure.js"></script>
````

or copy [federatedsecure.js](https://github.com/federatedsecure/client-javascript/blob/main/src/federatedsecure.js) to your website project.


# Example

The following HTML and Javascript code is a self-contained example that runs in a browser. It requires three Federated Secure Computing servers providing the [SIMON](https://github.com/federatedsecure/service-simon) protocol.

The small web application takes a list of numbers from each of the three parties (Alice, Bob, and Charlie) and computes some univariate statistics.

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Federated Secure Computing demo</title>
    </head>
    <body>
        <h1>Federated Secure Computing demo</h1>
        <p>server-side engine: <a href="https://github.com/federatedsecure/server">server</a></p>
        <p>protocol: <a href="https://github.com/federatedsecure/service-simon">SIMON</a></p>
        <p>microprotocol: <b>univariate statistics</b></p>
        <p>client-side language: <b>HTML / JavaScript</b></p>
        <form>
 
            <h2>step one: set up a private peer-to-peer-network</h2>
            <p>Alice's server: <label><input type="url" value="http://127.0.0.1:55501" id="input_url_alice"></label></p>
            <p>Bob's server: <label><input type="url" value="http://127.0.0.1:55502" id="input_url_bob"></label></p>
            <p>Charlie's server: <label><input type="url" value="http://127.0.0.1:55503" id="input_url_charlie"></label></p>
            <p>shared UUID of the VPN: <label><input type="url" value="a7f4fea2-b5f4-4d64-9705-45b3629a8145" id="input_uuid"></label></p>
 
 
            <h2>step two: select role of this instance</h2>
            <p><label><input type="radio" name="name_role" value="0" id="input_role_alice" checked="checked"></label> Alice</p>
            <p><label><input type="radio" name="name_role" value="1" id="input_role_bob"></label> Bob</p>
            <p><label><input type="radio" name="name_role" value="2" id="input_role_charlie"></label> Charlie</p>
            <p>
                <em>note: you need to open this page in three separate browser windows and select every role once.</em><br>
            </p>
 
            <h2>step three: enter secret private data of this instance</h2>
            <p><label><textarea rows="10" id="input_data">1&#10;2&#10;3&#10;4&#10;5&#10;6&#10;7&#10;8&#10;9&#10;10</textarea></label></p>
            <p><em>note: one number per line. number formatting based on your system and browser settings. leave default data or try integers first.</em><br></p>
 
        </form>
 
            <h2>step four: run the federated analysis</h2>
            <p><label><button id="btnCompute">get the result. keep your data.</button></label></p>
            <p><em>note: you need to press this button in each of the three browser windows once.</em><br></p>
 
            <h2>step five: result</h2>
            <div id="output_text">...</div>
 
    </body>
 
    <script src="https://cdn.jsdelivr.net/gh/federatedsecure/client-javascript/src/federatedsecure.js"></script>
    <script>
 
        const compute = async function () {
 
            document.querySelector("#output_text").innerText = "Running ...";
 
            // read options and input data from HTML form
            let nodes = [document.getElementById("input_url_alice").value,
                         document.getElementById("input_url_bob").value,
                         document.getElementById("input_url_charlie").value]
            let myself = parseInt(document.querySelector('input[name="name_role"]:checked').value);
            let data = document.getElementById("input_data").value.split('\n').map(
                function(item) { return parseInt(item); });
 
            // run the federated analysis
            let api = new Api({url: nodes[myself]});
            api.create([], {protocol: "Simon"})
                .then(microservice => {
                    microservice.attribute("compute")
                        .then(compute => {
                            compute.call([], {microprotocol: "StatisticsUnivariate",
                                              network: {nodes: nodes,
                                                        myself: myself,
                                                        uuid: document.getElementById("input_uuid").value},
                                              data: data})
                                .then(result => {
                                    api.download(result)
                                        .then(dl => {
                                            document.querySelector("#output_text").innerText = JSON.stringify(dl);
                                        })
                                })
                        })
                })
        };
 
        document.querySelector("#btnCompute").addEventListener("click", compute);
 
    </script>
 
</html>
```

Depending on the input, the output might read:

```json
{
  "inputs": 3,
  "result":
  {
    "coefficient_of_variation": 0.5311606738473833,
    "coefficient_of_variation_mle": 0.5222329678670935,
    "geometric_mean": 4.528728688116532,
    "harmonic_mean": 3.4141715214743513,
    "hyper_flatness": 3.7033976124885215,
    "hyper_skewness": 0,
    "kurtosis": 1.7757575757575756,
    "kurtosis_excess": -1.2242424242424244,
    "maximum": 10,
    "mean": 5.5,
    "minimum": 1,
    "root_mean_square": 6.2048368229954285,
    "root_mean_square_deviation": 2.8722813232690143,
    "samples": 30,
    "skewness": 0,
    "standard_deviation": 2.9213837061606083,
    "standard_deviation_mle": 2.8722813232690143,
    "standard_error_of_sample_mean": 0.5333692516640697,
    "sum": 165,
    "variance": 8.53448275862069,
    "variance_mle": 8.25,
    "variance_of_sample_mean": 0.28448275862068967
  }
}
```
