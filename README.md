# nodejs-express-sequelize-mysql
Tutorial membuat rest api dengan Node.js Express Sequelize dan MySQL

Cek artikel lengkapnya diwebsite kami [https://kreasify.com](https://kreasify.com/node-js-rest-api-express-sequelize-mysql/)

Express merupakan salah satu web framework paling populer untuk Node.js yang mendukung routing, middleware, view system.

Sedangkan Sequalize adalah promised-based Node.js ORM yang mendukung dialect Postgres, MySQL, SQL Server.

Pada tutorial ini, saya akan menunjukkan langkah-langkah untuk membangun Restful CRUD API Node.js menggunakan Express, Sequalize dengan database MySQL.

Untuk mengikuti tutorial ini, kamu harus install MySQL terlebih dahulu pada laptop kamu.

Node.js Rest CRUD API overview
------------------------------

Kita akan membuat Rest Api yang bisa create, retrieve, update, delete and find Tutorials by title.

Pertama, kita mulai dengan server web express.

Kemudian, kita tambahkan konfigurasi database MySQL, buat model dengan Sequalize, dan buat controller.

Selanjutnya kita definisikan routes untuk mengurus semua operasi CRUD (termasuk custom finder)

Tabel berikut menunjukkan tampilan Rest Api yang akan dikeluarkan:

| Methods | Urls | Actions |
| --- | --- | --- |
| GET | api/tutorials | get all Tutorials |
| GET | api/tutorials/:id | get Tutorial by `id` |
| POST | api/tutorials | add new Tutorial |
| PUT | api/tutorials/:id | update Tutorial by `id` |
| DELETE | api/tutorials/:id | remove Tutorial by `id` |
| DELETE | api/tutorials | remove all Tutorials |
| GET | api/tutorials/published | find all published Tutorials |
| GET | api/tutorials?title=\[kw\] | find all Tutorials which title contains `'kw'` |

selanjutnya, kita akan menguji dengan postman

struktur projeknya seperti ini:

 [![struktur](images/dev/express/node-js-express-sequelize-mysql-example-project-structure.png "struktur")](/uploads/images/dev/express/node-js-express-sequelize-mysql-example-project-structure.png) 

Buat Node.js App
----------------

pertama, kita buat folder

|     |     |
| --- | --- |
| 1<br>    2 | mkdir nodejs-express-sequelize-mysql<br>    cd nodejs-express-sequelize-mysql |

kemudian, kita inisialize node.js app dengan file `package.json`

 [![node-js-express-sequelize-mysql-package](images/dev/express/node-js-express-sequelize-mysql-package.png "node-js-express-sequelize-mysql-package")](/uploads/images/dev/express/node-js-express-sequelize-mysql-package.png) 

Kita perlu menginstal modul npm yang dibutuhkan: express, sequalize, mysql2, dan cors.

jalankan perintah berikut:

|     |     |
| --- | --- |
| 1   | npm install express sequelize mysql2 cors --save |

tampilan file `package.json` akan terlihat seperti ini:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18 | {<br>      "name": "bezkoder-nodejs-express-sequelize-mysql",<br>      "version": "1.0.0",<br>      "description": "",<br>      "main": "server.js",<br>      "scripts": {<br>        "dev": "nodemon server.js"<br>      },<br>      "keywords": [],<br>      "author": "",<br>      "license": "ISC",<br>      "dependencies": {<br>        "cors": "^2.8.5",<br>        "express": "^4.17.2",<br>        "mysql2": "^2.3.3",<br>        "sequelize": "^6.14.0"<br>      }<br>    } |

Setup Express web server
------------------------

Didalam folder root, buatlah file baru `server.js`

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18<br>    19<br>    20<br>    21<br>    22<br>    23<br>    24<br>    25<br>    26<br>    27 | const express = require("express");<br>    const cors = require("cors");<br>    const app = express();<br>    <br>    var corsOptions = {<br>      origin: "http://localhost:8081"<br>    };<br>    <br>    app.use(cors(corsOptions));<br>    <br>    // parse requests of content-type - application/json<br>    app.use(express.json());<br>    <br>    // parse requests of content-type - application/x-www-form-urlencoded<br>    app.use(express.urlencoded({ extended: true }));<br>    <br>    // simple route<br>    app.get("/", (req, res) => {<br>      res.json({ message: "Welcome to bezkoder application." });<br>    });<br>    <br>    // set port, listen for requests<br>    const PORT = process.env.PORT \| 8080;<br>    <br>    app.listen(PORT, () => {<br>      console.log(`Server is running on port ${PORT}.`);<br>    }); |

Apa yang kita lakukan yaitu:

* import modul express dan cors:
    
    _express_ digunakan untuk membangun Rest api
    
    _cors_ menyediakan Express middleware untuk mengaktifkan CORS dengan pilihan variasi
    
* buat aplikasi Express, kemudian tambahkan body-parser (json dan urlencoder) dan cors middlewares menggunakan app.use() method. notice that kita set origin http://localhost:8081
    
* definisikan GET route yang sederhana untuk test listen port 8080 untuk incoming requests
    

Selanjutnya mari kita ubah file `package.json` dengan menambahkan kode berikut pada baris ke 5:

|     |     |
| --- | --- |
| 1   | "main": "server.js", |

dan tambahkan kode berikut pada baris ke 7:

|     |     |
| --- | --- |
| 1   | "dev": "nodemon server.js", |

sehingga isi file `package.json` -nya menjadi seperti ini:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18 | {<br>      "name": "bezkoder-nodejs-express-sequelize-mysql",<br>      "version": "1.0.0",<br>      "description": "",<br>      "main": "server.js",<br>      "scripts": {<br>        "dev": "nodemon server.js"<br>      },<br>      "keywords": [],<br>      "author": "",<br>      "license": "ISC",<br>      "dependencies": {<br>        "cors": "^2.8.5",<br>        "express": "^4.17.2",<br>        "mysql2": "^2.3.3",<br>        "sequelize": "^6.14.0"<br>      }<br>    } |

Selanjutnya jalankan perintah berikut pada terminal:

dengan npm

|     |     |
| --- | --- |
| 1   | npm run dev |

atau dengan yarn

|     |     |
| --- | --- |
| 1   | yarn dev |

 [![01-jalankan-perintah-terminal](images/dev/express/01-jalankan-perintah-terminal.png "01-jalankan-perintah-terminal")](/uploads/images/dev/express/01-jalankan-perintah-terminal.png) 

Lalu buka port 8080 di web browser:

 [![01-buka-port-di-browser](images/dev/express/01-buka-port-di-browser.png "01-buka-port-di-browser")](/uploads/images/dev/express/01-buka-port-di-browser.png) 

Yeah, langkah pertama sudah selesai. Selanjutnya kita akan menggunakan Sequelize.

Konfigurasi database MySQL dan Sequelize
----------------------------------------

Didalam folder `app`, buatlah folder `config` terpisah untuk konfigurasi dengan file `dn.config.js` seperti ini:

 [![01-app-db-config-folder](images/dev/express/01-app-db-config-folder.png "01-app-db-config-folder")](/uploads/images/dev/express/01-app-db-config-folder.png) 

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13 | module.exports = {<br>      HOST: "localhost",<br>      USER: "root",<br>      PASSWORD: "",<br>      DB: "testdb",<br>      dialect: "mysql",<br>      pool: {<br>        max: 5,<br>        min: 0,<br>        acquire: 30000,<br>        idle: 10000<br>      }<br>    }; |

[![01-mysql-db-config](images/dev/express/01-mysql-db-config.png "01-mysql-db-config")](/uploads/images/dev/express/01-mysql-db-config.png)

Pada baris kode diatas, lima parameter pertama untuk koneksi MySQL.

pool (opsional), akan digunakan untuk koneksi Sequelize _pool configuration_:

* _max_: jumlah maksimum koneksi didalam pool
* _min_: jumlah minimum koneksi didalam pool
* _idle_: waktu maksimal, dalam _milliseconds_, menunjukkan waktu koneksi yang dapat di tunggu sebelum rilis
* _acquire_: waktu maksimal, dalam _milliseconds_, _pool_ itu akan mencoba mendapatkan koneksi sebelum menunjukkan kesalahan

Lebih detailnya, silakan kunjungi Referensi Api [Sequelize constructor](https://sequelize.org/master/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)

Inisialisasi Sequelize
----------------------

Kita akan menginisialisasi Sequelize di folder `app/models` yang akan berisi model di langkah berikutnya.

Buatlah folder baru didalam folder app dengan nama `models`, lalu buatlah file baru `index.js` didalamnya.

Tulislah kode dibawah ini kedalam file `index.js`.

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18<br>    19<br>    20<br>    21<br>    22<br>    23<br>    24 | const dbConfig = require("../config/db.config.js");<br>    <br>    const Sequelize = require("sequelize");<br>    const sequelize = new Sequelize(dbConfig.DB, dbConfig.USER, dbConfig.PASSWORD, {<br>      host: dbConfig.HOST,<br>      dialect: dbConfig.dialect,<br>      operatorsAliases: 0,<br>    <br>      pool: {<br>        max: dbConfig.pool.max,<br>        min: dbConfig.pool.min,<br>        acquire: dbConfig.pool.acquire,<br>        idle: dbConfig.pool.idle<br>      }<br>    });<br>    <br>    const db = {};<br>    <br>    db.Sequelize = Sequelize;<br>    db.sequelize = sequelize;<br>    <br>    db.tutorials = require("./tutorial.model.js")(sequelize, Sequelize);<br>    <br>    module.exports = db; |

 [![01-inisialisasi-sequelize](images/dev/express/01-inisialisasi-sequelize.png "01-inisialisasi-sequelize")](/uploads/images/dev/express/01-inisialisasi-sequelize.png) 

Jangan lupa untuk memanggil `sync()` didalam file `server.js`

|     |     |
| --- | --- |
| 1<br>    2<br>    3<br>    4<br>    5<br>    6 | const app = express();<br>    <br>    app.use(...);<br>    <br>    const db = require("./app/models");<br>    db.sequelize.sync(); |

Dalam proses development, kamu mungkin perlu untuk drop tabel yang tersedia dan _re-sync' database. Cukup menggunakan `force: true` seperti kode berikut:

|     |     |
| --- | --- |
| 1<br>    2<br>    3 | db.sequelize.sync({ force: true }).then(() => {<br>        console.log("Drop and re-sync db.");<br>      }); |

Menjadi seperti ini:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18<br>    19<br>    20<br>    21<br>    22<br>    23<br>    24<br>    25<br>    26<br>    27<br>    28<br>    29<br>    30<br>    31 | const express = require("express");<br>    const cors = require("cors");<br>    const app = express();<br>    <br>    var corsOptions = {<br>      origin: "http://localhost:8081"<br>    };<br>    <br>    app.use(cors(corsOptions));<br>    <br>    // parse requests of content-type - application/json<br>    app.use(express.json());<br>    <br>    // parse requests of content-type - application/x-www-form-urlencoded<br>    app.use(express.urlencoded({ extended: true }));<br>    <br>    const db = require("./app/models");<br>    <br>    db.sequelize.sync();<br>    <br>    // simple route<br>    app.get("/", (req, res) => {<br>      res.json({ message: "Welcome to bezkoder application." });<br>    });<br>    <br>    // set port, listen for requests<br>    const PORT = process.env.PORT \| 8080;<br>    <br>    app.listen(PORT, () => {<br>      console.log(`Server is running on port ${PORT}.`);<br>    }); |

Tentukan Model Sequelize
------------------------

Didalam folder models, buatlah file `tutorial.model.js` seperti ini:

 [![01-tentukan-model-sequelize](images/dev/express/01-tentukan-model-sequelize.png "01-tentukan-model-sequelize")](/uploads/images/dev/express/01-tentukan-model-sequelize.png) 

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15 | module.exports = (sequelize, Sequelize) => {<br>      const Tutorial = sequelize.define("tutorial", {<br>        title: {<br>          type: Sequelize.STRING<br>        },<br>        description: {<br>          type: Sequelize.STRING<br>        },<br>        published: {<br>          type: Sequelize.BOOLEAN<br>        }<br>      });<br>    <br>      return Tutorial;<br>    }; |

Model Sequelize ini merupakan tabel tutorial dalam database MySQL. Kolom-kolom ini akan dibuat secara otomatis: id, title, description, published, createdAt, updatedAt.

Setelah menginisialisasi Sequelize, kita tidak perlu menulis fungsi CRUD, Sequelize mendukung semuanya:

* buat Tutorial baru: `create(object)`
* cari Tutorial dengan `id`: `findByPk(id)`
* panggil semua Tutorial: `findAll()`
* update tutorial dengan id: update(data, where: { id: id })`
* hapus Tutorial: `destroy(where: { id: id })`
* hapus semua Tutorial: `destroy(where: {})`
* cari semua Tutorial dengan title: `findAll({ where: { title: ... } })`

Fungsi ini akan digunakan dalam Controller kita.

Kami dapat meningkatkan contoh dengan menambahkan Komentar untuk setiap Tutorial. Ini adalah _One-to-Many Relationship_ dan saya menulis tutorial untuk ini di: [Sequelize Associations: One-to-Many example – Node.js, MySQL](https://bezkoder.com/sequelize-associate-one-to-many/)

Atau Kamu dapat menambahkan Tag untuk setiap Tutorial dan menambahkan Tutorial ke Tag (_Many-to-Many Relationship_): [Sequelize Many-to-Many Association example with Node.js & MySQL](https://bezkoder.com/sequelize-associate-many-to-many/)

Buat Controller
---------------

Di dalam folder `app/controllers`, mari buat file `tutorial.controller.js` dengan fungsi CRUD ini:

 [![01-buat-controller](images/dev/express/01-buat-controller.png "01-buat-controller")](/uploads/images/dev/express/01-buat-controller.png) 

* create
* findAll
* findOne
* update
* delete
* deleteAll
* findAllPublished

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18<br>    19<br>    20<br>    21<br>    22<br>    23<br>    24<br>    25<br>    26<br>    27<br>    28<br>    29<br>    30<br>    31<br>    32<br>    33<br>    34<br>    35<br>    36<br>    37<br>    38 | const db = require("../models");<br>    const Tutorial = db.tutorials;<br>    const Op = db.Sequelize.Op;<br>    <br>    // Create and Save a new Tutorial<br>    exports.create = (req, res) => {<br>      <br>    };<br>    <br>    // Retrieve all Tutorials from the database.<br>    exports.findAll = (req, res) => {<br>      <br>    };<br>    <br>    // Find a single Tutorial with an id<br>    exports.findOne = (req, res) => {<br>      <br>    };<br>    <br>    // Update a Tutorial by the id in the request<br>    exports.update = (req, res) => {<br>      <br>    };<br>    <br>    // Delete a Tutorial with the specified id in the request<br>    exports.delete = (req, res) => {<br>      <br>    };<br>    <br>    // Delete all Tutorials from the database.<br>    exports.deleteAll = (req, res) => {<br>      <br>    };<br>    <br>    // Find all published Tutorials<br>    exports.findAllPublished = (req, res) => {<br>      <br>    }; |

Mari kita terapkan fungsi-fungsi ini.

Buat objek baru (Create)
------------------------

Buat dan Simpan Tutorial baru:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18<br>    19<br>    20<br>    21<br>    22<br>    23<br>    24<br>    25<br>    26<br>    27<br>    28 | exports.create = (req, res) => {<br>      // Validate request<br>      if (!req.body.title) {<br>        res.status(400).send({<br>          message: "Content can not be empty!"<br>        });<br>        return;<br>      }<br>    <br>      // Create a Tutorial<br>      const tutorial = {<br>        title: req.body.title,<br>        description: req.body.description,<br>        published: req.body.published ? req.body.published : false<br>      };<br>    <br>      // Save Tutorial in the database<br>      Tutorial.create(tutorial)<br>        .then(data => {<br>          res.send(data);<br>        })<br>        .catch(err => {<br>          res.status(500).send({<br>            message:<br>              err.message \| "Some error occurred while creating the Tutorial."<br>          });<br>        });<br>    }; |

Ambil Objek (Retrieve)
----------------------

Ambil semua Tutorial (dengan kondisi) atau temukan berdasarkan judul dari database:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15 | exports.findAll = (req, res) => {<br>      const title = req.query.title;<br>      var condition = title ? { title: { [Op.like]: `%${title}%` } } : null;<br>    <br>      Tutorial.findAll({ where: condition })<br>        .then(data => {<br>          res.send(data);<br>        })<br>        .catch(err => {<br>          res.status(500).send({<br>            message:<br>              err.message \| "Some error occurred while retrieving tutorials."<br>          });<br>        });<br>    }; |

Kami menggunakan `req.query.title` untuk mendapatkan string kueri dari Permintaan dan menganggapnya sebagai syarat untuk metode `findAll()`.

Update Objek
------------

Perbarui Tutorial yang diidentifikasi oleh `id` dalam permintaan:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18<br>    19<br>    20<br>    21<br>    22<br>    23 | exports.update = (req, res) => {<br>      const id = req.params.id;<br>    <br>      Tutorial.update(req.body, {<br>        where: { id: id }<br>      })<br>        .then(num => {<br>          if (num == 1) {<br>            res.send({<br>              message: "Tutorial was updated successfully."<br>            });<br>          } else {<br>            res.send({<br>              message: `Cannot update Tutorial with id=${id}. Maybe Tutorial was not found or req.body is empty!`<br>            });<br>          }<br>        })<br>        .catch(err => {<br>          res.status(500).send({<br>            message: "Error updating Tutorial with id=" + id<br>          });<br>        });<br>    }; |

Menghapus objek (Delete)
------------------------

Hapus Tutorial dengan `id` yang ditentukan:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18<br>    19<br>    20<br>    21<br>    22<br>    23 | exports.delete = (req, res) => {<br>      const id = req.params.id;<br>    <br>      Tutorial.destroy({<br>        where: { id: id }<br>      })<br>        .then(num => {<br>          if (num == 1) {<br>            res.send({<br>              message: "Tutorial was deleted successfully!"<br>            });<br>          } else {<br>            res.send({<br>              message: `Cannot delete Tutorial with id=${id}. Maybe Tutorial was not found!`<br>            });<br>          }<br>        })<br>        .catch(err => {<br>          res.status(500).send({<br>            message: "Could not delete Tutorial with id=" + id<br>          });<br>        });<br>    }; |

Hapus semua objek (Delete all)
------------------------------

Hapus semua Tutorial dari database:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15 | exports.deleteAll = (req, res) => {<br>      Tutorial.destroy({<br>        where: {},<br>        truncate: false<br>      })<br>        .then(nums => {<br>          res.send({ message: `${nums} Tutorials were deleted successfully!` });<br>        })<br>        .catch(err => {<br>          res.status(500).send({<br>            message:<br>              err.message \| "Some error occurred while removing all tutorials."<br>          });<br>        });<br>    }; |

Temukan semua objek berdasarkan kondisi
---------------------------------------

Temukan semua Tutorial dengan `published = true`:

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12 | exports.findAllPublished = (req, res) => {<br>      Tutorial.findAll({ where: { published: true } })<br>        .then(data => {<br>          res.send(data);<br>        })<br>        .catch(err => {<br>          res.status(500).send({<br>            message:<br>              err.message \| "Some error occurred while retrieving tutorials."<br>          });<br>        });<br>    }; |

Kontroler ini dapat dimodifikasi sedikit untuk mengembalikan respons pagination:

|     |     |
| --- | --- |
| 1<br>    2<br>    3<br>    4<br>    5<br>    6 | {<br>        "totalItems": 8,<br>        "tutorials": [...],<br>        "totalPages": 3,<br>        "currentPage": 1<br>    } |

Anda dapat menemukan detail lebih lanjut di: [Server side Pagination in Node.js with Sequelize and MySQL](https://bezkoder.com/node-js-sequelize-pagination-mysql/)

Tentukan Rute (Define Routes)
-----------------------------

Ketika klien mengirim permintaan untuk titik akhir (_endpoint_) menggunakan permintaan HTTP (GET, POST, PUT, DELETE), kita perlu menentukan bagaimana server akan merespons dengan mengatur rute.

Ini adalah rute kami:

* `/api/tutorials`: GET, POST, DELETE
* `/api/tutorials/:id`: GET, PUT, DELETE
* `/api/tutorials/published`: GET

Buat `turorial.routes.js` di dalam folder `app/routes` dengan konten seperti ini:

 [![01-tentukan-rute-folder](images/dev/express/01-tentukan-rute-folder.png "01-tentukan-rute-folder")](/uploads/images/dev/express/01-tentukan-rute-folder.png) 

|     |     |
| --- | --- |
| 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10<br>    11<br>    12<br>    13<br>    14<br>    15<br>    16<br>    17<br>    18<br>    19<br>    20<br>    21<br>    22<br>    23<br>    24<br>    25<br>    26<br>    27<br>    28 | module.exports = app => {<br>      const tutorials = require("../controllers/tutorial.controller.js");<br>    <br>      var router = require("express").Router();<br>    <br>      // Create a new Tutorial<br>      router.post("/", tutorials.create);<br>    <br>      // Retrieve all Tutorials<br>      router.get("/", tutorials.findAll);<br>    <br>      // Retrieve all published Tutorials<br>      router.get("/published", tutorials.findAllPublished);<br>    <br>      // Retrieve a single Tutorial with id<br>      router.get("/:id", tutorials.findOne);<br>    <br>      // Update a Tutorial with id<br>      router.put("/:id", tutorials.update);<br>    <br>      // Delete a Tutorial with id<br>      router.delete("/:id", tutorials.delete);<br>    <br>      // Delete all Tutorials<br>      router.delete("/", tutorials.deleteAll);<br>    <br>      app.use('/api/tutorials', router);<br>    }; |

Anda dapat melihat bahwa kami menggunakan pengontrol dari `/controllers/tutorial.controller.js`.

Kita juga perlu menyertakan rute di `server.js` (tepat sebelum `app.listen()`):

|     |     |
| --- | --- |
| 1<br>    2<br>    3<br>    4<br>    5<br>    6<br>    7 | ...<br>    <br>    require("./app/routes/turorial.routes")(app);<br>    <br>    // set port, listen for requests<br>    const PORT = ...;<br>    app.listen(...); |

Uji API (Test Api)
------------------

Jalankan aplikasi Node.js kita dengan perintah: `node server.js`.

Konsol menunjukkan:

|     |     |
| --- | --- |
| 1<br>    2<br>    3<br>    4<br>    5 | Server is running on port 8080.<br>    Executing (default): DROP TABLE IF EXISTS `tutorials`;<br>    Executing (default): CREATE TABLE IF NOT EXISTS `tutorials` (`id` INTEGER NOT NULL auto_increment , `title` VARCHAR(255), `description` VARCHAR(255), `published` TINYINT(1), `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB;<br>    Executing (default): SHOW INDEX FROM `tutorials`<br>    Drop and re-sync db. |

Menggunakan Hoppscotch, kita akan menguji semua Api di atas.

1.  Buat Tutorial baru menggunakan `POST/tutorials` Api
    
     [![01-tes-buat-tutorial-baru](images/dev/express/01-tes-buat-tutorial-baru.png "01-tes-buat-tutorial-baru")](/uploads/images/dev/express/01-tes-buat-tutorial-baru.png) 
    
    Setelah membuat beberapa Tutorial baru, Anda dapat memeriksa tabel MySQL:
    
    |     |     |
    | --- | --- |
    | 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10 | mysql> select * from tutorials;<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+<br>    \| id \| title             \| description       \| published \| createdAt           \| updatedAt           \|<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+<br>    \|  1 \| JS: Node Tut #1   \| Tut#1 Description \|         0 \| 2019-12-13 01:13:57 \| 2019-12-13 01:13:57 \|<br>    \|  2 \| JS: Node Tut #2   \| Tut#2 Description \|         0 \| 2019-12-13 01:16:08 \| 2019-12-13 01:16:08 \|<br>    \|  3 \| JS: Vue Tut #3    \| Tut#3 Description \|         0 \| 2019-12-13 01:16:24 \| 2019-12-13 01:16:24 \|<br>    \|  4 \| Vue Tut #4        \| Tut#4 Description \|         0 \| 2019-12-13 01:16:48 \| 2019-12-13 01:16:48 \|<br>    \|  5 \| Node & Vue Tut #5 \| Tut#5 Description \|         0 \| 2019-12-13 01:16:58 \| 2019-12-13 01:16:58 \|<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+ |
    
2.  Ambil semua Tutorial menggunakan `GET /tutorials` Api
    
     [![01-tes-ambil-semua-tutorial](images/dev/express/01-tes-ambil-semua-tutorial.png "01-tes-ambil-semua-tutorial")](/uploads/images/dev/express/01-tes-ambil-semua-tutorial.png) 
    
3.  Ambil satu Tutorial dengan id menggunakan `GET /tutorials/:id` Api
    
     [![01-tes-ambil-tutorial-tunggal](images/dev/express/01-tes-ambil-tutorial-tunggal.png "01-tes-ambil-tutorial-tunggal")](/uploads/images/dev/express/01-tes-ambil-tutorial-tunggal.png) 
    
4.  Perbarui Tutorial menggunakan `PUT /tutorials/:id`
    
     [![01-tes-update-tutorial-tunggal](images/dev/express/01-tes-update-tutorial-tunggal.png "01-tes-update-tutorial-tunggal")](/uploads/images/dev/express/01-tes-update-tutorial-tunggal.png) 
    
    Periksa tabel `tutorials` setelah beberapa baris diperbarui:
    
    |     |     |
    | --- | --- |
    | 1<br>     2<br>     3<br>     4<br>     5<br>     6<br>     7<br>     8<br>     9<br>    10 | mysql> select * from tutorials;<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+<br>    \| id \| title             \| description       \| published \| createdAt           \| updatedAt           \|<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+<br>    \|  1 \| JS: Node Tut #1   \| Tut#1 Description \|         0 \| 2019-12-13 01:13:57 \| 2019-12-13 01:13:57 \|<br>    \|  2 \| JS: Node Tut #2   \| Tut#2 Description \|         0 \| 2019-12-13 01:16:08 \| 2019-12-13 01:16:08 \|<br>    \|  3 \| JS: Vue Tut #3    \| Tut#3 Description \|         1 \| 2019-12-13 01:16:24 \| 2019-12-13 01:22:51 \|<br>    \|  4 \| Vue Tut #4        \| Tut#4 Description \|         1 \| 2019-12-13 01:16:48 \| 2019-12-13 01:25:28 \|<br>    \|  5 \| Node & Vue Tut #5 \| Tut#5 Description \|         1 \| 2019-12-13 01:16:58 \| 2019-12-13 01:25:30 \|<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+ |
    
5.  Temukan semua Tutorial yang judulnya berisi `node`: `GET /tutorials?title=node`
    
     [![01-tes-ambil-semua-tutorial-dengan-id-khusus](images/dev/express/01-tes-ambil-semua-tutorial-dengan-id-khusus.png "01-tes-ambil-semua-tutorial-dengan-id-khusus")](/uploads/images/dev/express/01-tes-ambil-semua-tutorial-dengan-id-khusus.png) 
    
6.  Temukan semua Tutorial yang diterbitkan menggunakan `GET /tutorials/published` Api
    
     [![01-tes-ambil-semua-tutorial-dengan-kueri-published](images/dev/express/01-tes-ambil-semua-tutorial-dengan-kueri-published.png "01-tes-ambil-semua-tutorial-dengan-kueri-published")](/uploads/images/dev/express/01-tes-ambil-semua-tutorial-dengan-kueri-published.png) 
    
7.  Hapus Tutorial menggunakan `DELETE /tutorials/:id` Api
    
     [![01-tes-hapus-tutorial](images/dev/express/01-tes-hapus-tutorial.png "01-tes-hapus-tutorial")](/uploads/images/dev/express/01-tes-hapus-tutorial.png) 
    
    Tutorial dengan id=2 telah dihapus dari tabel `tutorials`:
    
    |     |     |
    | --- | --- |
    | 1<br>    2<br>    3<br>    4<br>    5<br>    6<br>    7<br>    8<br>    9 | mysql> select * from tutorials;<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+<br>    \| id \| title             \| description       \| published \| createdAt           \| updatedAt           \|<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+<br>    \|  1 \| JS: Node Tut #1   \| Tut#1 Description \|         0 \| 2019-12-13 01:13:57 \| 2019-12-13 01:13:57 \|<br>    \|  3 \| JS: Vue Tut #3    \| Tut#3 Description \|         1 \| 2019-12-13 01:16:24 \| 2019-12-13 01:22:51 \|<br>    \|  4 \| Vue Tut #4        \| Tut#4 Description \|         1 \| 2019-12-13 01:16:48 \| 2019-12-13 01:25:28 \|<br>    \|  5 \| Node & Vue Tut #5 \| Tut#5 Description \|         1 \| 2019-12-13 01:16:58 \| 2019-12-13 01:25:30 \|<br>    +----+-------------------+-------------------+-----------+---------------------+---------------------+ |
    
8.  Hapus semua Tutorial menggunakan `DELETE /tutorials` Api
    
     [![01-tes-hapus-semua-tutorial](images/dev/express/01-tes-hapus-semua-tutorial.png "01-tes-hapus-semua-tutorial")](/uploads/images/dev/express/01-tes-hapus-semua-tutorial.png) 
    
    Sekarang tidak ada baris dalam tabel `tutorials`:
    
    |     |     |
    | --- | --- |
    | 1<br>    2 | mysql> SELECT * FROM tutorials;<br>    Empty set (0.00 sec) |
    

Anda dapat menggunakan [Klien HTTP Sederhana menggunakan Axios](https://www.bezkoder.com/axios-request/) untuk memeriksanya.

Kesimpulan
----------

Hari ini, kita telah mempelajari cara membuat Node.js Rest Api dengan server web Express. Kami juga tahu cara menambahkan konfigurasi untuk database MySQL & Sequelize, membuat Model Sequelize, menulis pengontrol dan menentukan rute untuk menangani semua operasi CRUD.

Anda dapat menemukan hal yang lebih menarik di tutorial berikutnya: [– Pagination sisi server di Node.js dengan Sequelize dan MySQL](https://bezkoder.com/node-js-sequelize-pagination-mysql/)

_Return_ data pagination sebagai _response_:

|     |     |
| --- | --- |
| 1<br>    2<br>    3<br>    4<br>    5<br>    6 | {<br>        "totalItems": 8,<br>        "tutorials": [...],<br>        "totalPages": 3,<br>        "currentPage": 1<br>    } |

### Tags:

[Express](/tag/express/) [Sequelize](/tag/sequelize/) [MySQL](/tag/mysql/) [Rest Api](/tag/rest-api/)

### Bagikan:

[Non Minor, Inquit, Voluptas Percipitur Ex Vilissimis](http://localhost:1313/non-minor-inquit-voluptas-percipitur-ex-vilissimis/ "Previous page in dev")

[### Akhlis](/author/akhlis/)

Dream - Berhijab membuat Citra Kirana semakin modis. Artis yang akrab disapa Ciki itu mantap berhijab

### Artikel Terkait
