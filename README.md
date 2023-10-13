# Web GIS Tutorial

Project Application Name: **Peta Lokasi Objek**

Framework: **CodeIgniter 4**

Database: **PostgreSQL - PostGIS (recommended)** or **MySQL**

Presentasi: [https://docs.google.com/presentation/d/1nZsY9Y4Jz-MZ5HdeYoY5X9srUI4PL8ZyMOkudBR8Jy4/edit?usp=sharing](https://docs.google.com/presentation/d/1nZsY9Y4Jz-MZ5HdeYoY5X9srUI4PL8ZyMOkudBR8Jy4/edit?usp=sharing)

---

## CodeIgniter Documentation

[https://codeigniter.com/user_guide/index.html](https://codeigniter.com/user_guide/index.html)

## Install CodeIgniter 4 Menggunakan Composer

```
composer create-project codeigniter4/appstarter peta-lokasi-objek
```

## CI4 Database Connection

**PostgreSQL**

```
database.default.hostname = localhost
database.default.database = [_YOUR_DATABASE_NAME_]
database.default.username = postgres
database.default.password = [_YOUR_PASSWORD_POSTGRES_]
database.default.DBDriver = Postgre
database.default.DBPrefix =
database.default.port = 5432
```

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

## Membuat Model Datalokasiobjek

```
php spark make:model DatalokasiobjekModel
```

## Konfigurasi Model

```
protected $allowedFields = [
  'geom',
  'nama',
  'deskripsi',
  'latitude',
  'longitude',
];

// Dates
protected $useTimestamps = true;

// Add this method
public function dataobjek()
{
  // Query Builder
  $query = $this->select('id, ST_AsGeoJSON(geom) as geom, nama, deskripsi, created_at, updated_at')
    ->get()
    ->getResultArray();

  return $query;
}
```

## Membuat Migration Tabel Datalokasiobjek

```
php spark make:migration Datalokasiobjek
```

## Konfigurasi Migration Datalokasiobjek

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
  'deskripsi' => [
    'type' => 'TEXT',
    'null' => true,
  ],
  'latitude' => [
    'type' => 'decimal',
    'constraint' => '10,8',
  ],
  'longitude' => [
    'type' => 'decimal',
    'constraint' => '11,8',
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
$this->forge->createTable('datalokasiobjeks');
```

**public function down()**

```
$this->forge->dropTable('datalokasiobjeks');
```

## Membuat Seeder Datalokasiobjek

```
php spark make:seeder DatalokasiobjekSeeder
```

## Konfigurasi Seeder Datalokasiobjek

```
// get data from json file at public/data
$json = file_get_contents('data/datalokasiobjek.json');

// decode json
$data = json_decode($json, true);

// insert to database
foreach ($data as $value) {
  $this->db->table('datalokasiobjeks')->insert(
    [
      'geom' => "POINT(" . $value['longitude'] . " " . $value['latitude'] . ")", // for postgis
      'nama' => $value['nama'],
      'deskripsi' => $value['deskripsi'],
      'latitude' => $value['latitude'],
      'longitude' => $value['longitude'],
      'created_at' => date('Y-m-d H:i:s'),
      'updated_at' => date('Y-m-d H:i:s'),
    ]
  );
}
```

> Download JSON Data Sample: [Direct Link](data/datalokasiobjek.json) | [Google Drive](https://drive.google.com/file/d/1Jfnn3Y6bhvy6sye55_kxpnFI5NMl_vk-/view?usp=sharing)

## Controller - Method GeoJSON Point Services

```
$datalokasiobjek = $this->datalokasiobjek->dataobjek();

$geojson = [
  'type' => 'FeatureCollection',
  'features' => [],
];
foreach ($datalokasiobjek as $row) {
  $feature = [
    'type' => 'Feature',
    'properties' => $row,
    'geometry' => json_decode($row['geom']),
  ];
  
  // make hidden geom
  unset($feature['properties']['geom']);

  array_push($geojson['features'], $feature);
}

// json numeric check
$geojson = json_encode($geojson, JSON_NUMERIC_CHECK);
return $this->response->setJSON($geojson);
```

## View - Leaflet Point GeoJSON Layer with jQuery

```
// GeoJSON Point Layer
var point = L.geoJson(null, {
  onEachFeature: function (feature, layer) {
    var popupContent = "<h5>" + feature.properties.nama + "</h5>" +
      "<p>" + feature.properties.deskripsi + "</p>";
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

[https://anshori.github.io/leaflet-search-coordinates/](https://anshori.github.io/leaflet-search-coordinates/)

---

> [unsorry@2023](https://unsorry.net)
