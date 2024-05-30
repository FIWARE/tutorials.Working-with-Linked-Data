[![FIWARE Banner](https://fiware.github.io/tutorials.Working-with-Linked-Data/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Working-with-Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/)

This tutorial teaches FIWARE users how to architect and design a system based on **linked data** and to alter linked
data context programmatically. The tutorial extends the knowledge gained from the equivalent
[NGSI-v2 tutorial](https://github.com/FIWARE/tutorials.Accessing-Context/) and enables a user understand how to write
code in an [NGSI-LD](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf) capable
[Node.js](https://nodejs.org/) [Express](https://expressjs.com/) application in order to retrieve and alter context
data. This removes the need to use the command-line to invoke cUrl commands.

The tutorial is mainly concerned with discussing code written in Node.js, however some of the results can be checked by
making [cUrl](https://ec.haxx.se/) commands.
[Postman documentation](https://www.postman.com/downloads/) for the same commands is also
available.

# Start-Up

## NGSI-LD Smart Supermarket

**NGSI-LD** offers JSON-LD based interoperability used for Federations and Data Spaces. To run this **NGSI-LD** tutorial for **NGSI-v2** developers, use the `NGSI-v2` branch.

```console
git clone https://github.com/FIWARE/tutorials.Working-with-Linked-Data.git
cd tutorials.Working-with-Linked-Data
git checkout NGSI-v2

./services create
./services start
```

| [![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf) | :books: [Documentation](https://github.com/FIWARE/tutorials.Working-with-Linked-Data/tree/NGSI-v2) | <img src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.Working-with-Linked-Data/) |
| --- | --- | --- |

---

## License

[MIT](LICENSE) © 2020-2024 FIWARE Foundation e.V.
