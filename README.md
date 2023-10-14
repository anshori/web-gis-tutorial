# Web GIS Tutorial

Project Application Name: **Peta Lokasi Mahasiswa**

Framework: **CodeIgniter 4**

Database: **MySQL**

Presentasi: [https://docs.google.com/presentation/d/1SzhhnJqGAyvIsACKlFeBsbWa3FUTVFanpxpNelYCsHo/edit?usp=sharing](https://docs.google.com/presentation/d/1SzhhnJqGAyvIsACKlFeBsbWa3FUTVFanpxpNelYCsHo/edit?usp=sharing)

---

## CodeIgniter Documentation
[https://codeigniter.com/user_guide/index.html](https://codeigniter.com/user_guide/index.html)


## Install CodeIgniter 4 Menggunakan Composer
```
composer create-project codeigniter4/appstarter peta-lokasi-mahasiswa
```

## CI4 Database Connection
**MySQL**
```
database.default.hostname = localhost
database.default.database = [_YOUR_DATABASE_NAME_]
database.default.username = root
database.default.password = [_YOUR_PASSWORD_ROOT_]
database.default.DBDriver = MySQLi
database.default.DBPrefix =
database.default.port = 3306
```

## Membuat Migration Tabel Datamahasiswa
```
php spark make:migration Datamahasiswa
```

## Konfigurasi Migration Datamahasiswa
**public function up()**
```
$this->forge->addField([
  'id' => [
    'type' => 'INT',
    'constraint' => 11,
    'auto_increment' => true,
  ],
  'geom' => [
    'type' => 'geometry',
  ],
  'nama' => [
    'type' => 'VARCHAR',
    'constraint' => 255,
  ],
  'jeniskelamin' => [
    'type' => 'VARCHAR',
    'constraint' => 255,
  ],
  'alamat' => [
    'type' => 'TEXT',
    'null' => true,
  ],
  'created_at' => [
    'type' => 'DATETIME',
    'null' => true,
  ],
  'updated_at' => [
    'type' => 'DATETIME',
    'null' => true,
  ],
]);
$this->forge->addKey('id', true);
$this->forge->createTable('datamahasiswas');
```

**public function down()**
```
$this->forge->dropTable('datamahasiswas');
```

## Membuat Seeder Datamahasiswa
```
php spark make:seeder DatamahasiswaSeeder
```

## Konfigurasi Seeder Datamahasiswa
```
// get data from json file at public/data
$json = file_get_contents('data/datamahasiswa.json');

// decode json
$data = json_decode($json, true);

// insert to database
foreach ($data as $value) {
  $sqlquery = "INSERT INTO datamahasiswas (geom, nama, jeniskelamin, alamat, created_at, updated_at) VALUES (ST_GeomFromText('POINT(" . $value['longitude'] . " " . $value['latitude'] . ")'), '" . $value['nama'] . "', '" . $value['jeniskelamin'] . "', '" . $value['alamat'] . "', '" . date('Y-m-d H:i:s') . "', '" . date('Y-m-d H:i:s') . "')";
  $this->db->query($sqlquery);
}
```

> Download JSON Data Sample: [Direct Link](data/datamahasiswa.json) | [Google Drive](https://drive.google.com/file/d/1wJskLFT0fjuy9Xh3hxLcPn-RmNG-lHb4/view?usp=sharing)


## Membuat Model Datamahasiswa
```
php spark make:model DatamahasiswaModel
```

## Konfigurasi Model
```
protected $allowedFields = [
  'geom',
  'nama',
  'jeniskelamin',
  'alamat',
];

// Dates
protected $useTimestamps = true;
```

## Method query data mahasiswa di dalam model
```
// Query data mahasiswa
public function mahasiswa()
{
  // Query Builder
  $query = $this->select('id, ST_AsGeoJSON(geom) as geom, nama, jeniskelamin, alamat, created_at, updated_at')
    ->get()
    ->getResultArray();

  return $query;
}
```

## Method query data mahasiswa berdasarkan ID di dalam model
```
// Query data mahasiswa by id mahasiswa
public function mahasiswabyid()
{
  // Query Builder
  $query = $this->select('id, ST_AsGeoJSON(geom) as geom, nama, jeniskelamin, alamat, created_at, updated_at')
    ->where('id', $id)
    ->first();

  return $query;
}
```

## Controller Home
```
public function __construct()
{
  $this->datamhs = new Datamahasiswa();
}

// method untuk menampilkan halaman peta
public function index(): string
{
  $data = [
    'title' => 'Peta Lokasi Mahasiswa',
  ];

  return view('map', $data);
}
```

## Controller - Method GeoJSON Point Services
```
public function geojson_point(): object
{
  $datamhs = $this->datamhs->mahasiswa();

  $geojson = [
    'type' => 'FeatureCollection',
    'features' => [],
  ];

  foreach ($datamhs as $row) {
    $feature = [
      'type' => 'Feature',
      'properties' => $row,
      'geometry' => json_decode($row['geom']),
    ];

    // hidden geom from properties
    unset($feature['properties']['geom']);

    array_push($geojson['features'], $feature);
  }

  // json numeric check
  $geojson = json_encode($geojson, JSON_NUMERIC_CHECK);
  return $this->response->setJSON($geojson);
}
```

## Route
```
// call method index from controller home
$routes->get('/', 'Home::index');

// call method geojson_point from controller home
$routes->get('geojson-point', 'Home::geojson_point');
```

## View - Leaflet Map with Navbar Bootstrap
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="shortcut icon" href="https://unsorry.net/assets-date/images/favicon.png" type="image/x-icon">
    <title>Peta Lokasi Mahasiswa</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
    <style>
      #map {
        margin-top: 55px;
        height: calc(100vh - 55px);
        width: 100%;
      }
    </style>
  </head>

  <body>
    <!-- Navbar Bootstap -->
    <nav class="navbar navbar-expand-lg bg-dark border-bottom border-body fixed-top" data-bs-theme="dark">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">Peta Lokasi Mahasiswa</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav ms-auto">
            <li class="nav-item">
              <a class="nav-link" href="#">Peta</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">Data</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">Tambah Data</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#" data-bs-toggle="modal" data-bs-target="#infoModal">Info</a>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <div id="map"></div>

    <!-- Modal Info -->
    <div class="modal fade" id="infoModal" tabindex="-1" aria-labelledby="infoModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h1 class="modal-title fs-5" id="infoModalLabel">Info</h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <p>Aplikasi ini dibuat dengan menggunakan PHP framework CodeIgniter 4 dan database MySQL dalam rangka kegiatan Praktisi Mengajar di Fakultas Geografi, Universitas Muhammadiyah Surakarta (UMS)</p>
            <p class="mt-4 text-end text-secondary"><small>unsorry@2023</small></p>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Tutup</button>
          </div>
        </div>
      </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
    <script>
      // init map
      var map = L.map('map').setView([-7.7956, 110.3695], 10);

      // init basemap
      var basemap = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 18,
        attribution: 'unsorry@2023',
      });

      // add basemap to map
      // basemap.addTo(map);

      // scale bar
      L.control.scale({
        position: 'bottomleft',
        metric: true,
        imperial: false,
      }).addTo(map);
    </script>
  </body>
</html>
```

Untuk menentukan koordinat tengah peta dan nilai zoom level peta, Anda dapat menggunakan aplikasi ini
>[https://anshori.github.io/leafletjs-mapcentercoordinate](https://anshori.github.io/leafletjs-mapcentercoordinate)


## View - Leaflet Point GeoJSON Layer with jQuery
```
// GeoJSON Point Layer
var point = L.geoJson(null, {
  onEachFeature: function (feature, layer) {
    var popupContent = "<h5>" + feature.properties.nama + "</h5>" +
      "<p>" + feature.properties.jeniskelamin + "</p>"
      "<p>" + feature.properties.alamat + "</p>";
    layer.on({
      click: function (e) {
        point.bindPopup(popupContent);
      },
      mouseover: function (e) {
        point.bindTooltip(feature.properties.nama);
      },
    });
  },
});
$.getJSON("<?= base_url(‘geojson-point’) ?>", function (data) {
  point.addData(data);
  map.addLayer(point);
  // fit map to geojson
  map.fitBounds(point.getBounds());
});
```

## View - Form Input With Coordinate by Marker Location
[https://anshori.github.io/leaflet-search-coordinates/](https://anshori.github.io/leaflet-search-coordinates/)

---

> [unsorry@2023](https://unsorry.net)
