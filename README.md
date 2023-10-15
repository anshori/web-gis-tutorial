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

## Menjalankan Migration
```
php spark migrate --all
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

## Menjalankan Seeder
```
php spark db:seed DatamahasiswaSeeder
```

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
public function mahasiswabyid($id)
{
  // Query Builder
  $query = $this->select('id, ST_AsGeoJSON(geom) as geom, nama, jeniskelamin, alamat, created_at, updated_at')
    ->where('id', $id)
    ->first();

  return $query;
}
```

## Controller Home - Construct
```
public function __construct()
{
  $this->datamhs = new DatamahasiswaModel();
}
```

## Controller Home - Index
```
// method index untuk menampilkan peta sebagai landing page
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
  // Memanggil data mahasiswa dari method mahasiswa pada model DatamahasiswaModel
  $datamhs = $this->datamhs->mahasiswa();

  // Membuat variabel geojson
  $geojson = [
    'type' => 'FeatureCollection',
    'features' => [],
  ];

  // perulangan/looping data mahasiswa
  foreach ($datamhs as $row) {
    $feature = [
      'type' => 'Feature',
      'properties' => $row,
      'geometry' => json_decode($row['geom']),
    ];

    // hidden geom from properties
    unset($feature['properties']['geom']);

    // memasukkan value dari variabel $feature ke dalam objek features di dalam variabel geojson
    array_push($geojson['features'], $feature);
  }

  // json numeric check
  $geojson = json_encode($geojson, JSON_NUMERIC_CHECK);

  // mengeluarkan semua value dalam bentuk json
  return $this->response->setJSON($geojson);
}
```

## Routes
```
// call method index from controller home
$routes->get('/', 'Home::index');

// call method geojson_point from controller home
$routes->get('geojson-point', 'Home::geojson_point');
```

## View - Leaflet Map
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="shortcut icon" href="https://unsorry.net/assets-date/images/favicon.png" type="image/x-icon">
    <title>Peta Lokasi Mahasiswa</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
    <style>
      #map {
        width: 100%;
        height: 100vh;
      }
    </style>
  </head>

  <body>
    <div id="map"></div>

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

### Navbar Bootstrap
> [https://getbootstrap.com/docs/5.3/components/navbar/#nav](https://getbootstrap.com/docs/5.3/components/navbar/#nav)

## View - Leaflet Point GeoJSON Layer with jQuery
```
// GeoJSON Point Layer
var point = L.geoJson(null, {
  onEachFeature: function (feature, layer) {
    var popupContent = "<h5>" + feature.properties.nama + "</h5>" +
      "<p>" + feature.properties.jeniskelamin + "</p>" +
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
$.getJSON("<?= base_url('geojson-point') ?>", function (data) {
  point.addData(data);
  map.addLayer(point);
  // fit map to geojson
  map.fitBounds(point.getBounds());
});
```

## View - Form Input With Coordinate by Marker Location
> [https://anshori.github.io/leaflet-search-coordinates/](https://anshori.github.io/leaflet-search-coordinates/)

## Controller - Create Data
```
public function create(): string
{
  return view('tambah-data');
}
```

## Create Route
```
$routes->get('tambah-data', 'Home::create');
```

## Controller - Store Data
```
public function store(): object
{
  // time zone jakarta
  date_default_timezone_set('Asia/Jakarta');

  // insert data
  $sqlquery = "INSERT INTO datamahasiswas (geom, nama, jeniskelamin, alamat, created_at, updated_at)
    VALUES (
      ST_GeomFromText('POINT(" . $_POST['longitude'] . " " . $_POST['latitude'] . ")'), '" .
      $_POST['nama'] . "', '" .
      $_POST['jeniskelamin'] . "', '" .
      $_POST['alamat'] . "', '" .
      date('Y-m-d H:i:s') . "', '" .
      date('Y-m-d H:i:s') .
    "')";

  $this->db->query($sqlquery);

  return redirect()->to(base_url('/'));
}
```

## Store Route
```
$routes->post('store', 'Home::store');
```

## Controller - Edit Data
```
public function edit($id): string
{
  // get data mahasiswa by id
  $datamhs = $this->datamhs->mahasiswabyid($id);

  // convert geojson geometry to array
  $coordinates = json_decode($datamhs['geom'], true)['coordinates'];

  $data = [
    'datamhs' => $datamhs,
    'longitude' => $coordinates[0],
    'latitude' => $coordinates[1],
  ];

  return view('edit-data', $data);
}
```

## Edit Route
```
$routes->get('edit-data/(:num)', 'Home::edit/$1');
```

## Controller - Update Data
```
public function update($id): object
{
  // time zone jakarta
  date_default_timezone_set('Asia/Jakarta');

  // update data
  $sqlquery = "UPDATE datamahasiswas SET
    geom = ST_GeomFromText('POINT(" . $_POST['longitude'] . " " . $_POST['latitude'] . ")'), " .
    "nama = '" . $_POST['nama'] . "', " .
    "jeniskelamin = '" . $_POST['jeniskelamin'] . "', " .
    "alamat = '" . $_POST['alamat'] . "', " .
    "updated_at = '" . date('Y-m-d H:i:s') . "'" .
    "WHERE id = '" . $id . "'";

  $this->db->query($sqlquery);

  return redirect()->to(base_url('tabel'));
}
```

## Update Route
```
$routes->put('update/(:num)', 'Home::update/$1');
```

## Controller - Delete Data
```
public function delete($id): object
{
  $this->datamhs->delete($id);

  return redirect()->to(base_url('tabel'));
}
```

## Delete Route
```
$routes->delete('delete/(:num)', 'Home::delete/$1');
```

---

> [unsorry@2023](https://unsorry.net)
