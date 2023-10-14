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
    'type' => 'geography(geometry,4326)',
  ],
  'nama' => [
    'type' => 'VARCHAR',
    'constraint' => 255,
  ],
  'jeniskelamin' => [
    'type' => 'TEXT',
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
