# Web GIS Tutorial

Project Application Name: **Peta Lokasi Objek**

Framework: **CodeIgniter 4**

Presentasi: [https://docs.google.com/presentation/d/1nZsY9Y4Jz-MZ5HdeYoY5X9srUI4PL8ZyMOkudBR8Jy4/edit?usp=sharing](https://docs.google.com/presentation/d/1nZsY9Y4Jz-MZ5HdeYoY5X9srUI4PL8ZyMOkudBR8Jy4/edit?usp=sharing)

___    

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
  'nama',
  'deskripsi',
  'latitude',
  'longitude',
];

// Dates
protected $useTimestamps = true;
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
// get data from json file
$json = file_get_contents('data/datalokasiobjek.json');

// decode json
$data = json_decode($json, true);

// insert to database
$this->db->table('datalokasiobjeks')->insertBatch($data);
```

> [Download json data](https://drive.google.com/file/d/1SmIKSe52Xyg5hZjKaGhhLgQfx9iLvMtG/view?usp=sharing)


## Controller - Method GeoJSON Point Services
```
$datalokasiobjek = $this->datalokasiobjek->findAll();

$geojson = [
   'type' => 'FeatureCollection',
   'features' => [],
];
foreach ($datalokasiobjek as $row) {
   $feature = [
       'type' => 'Feature',
       'properties' => $row,
       'geometry' => [
           'type' => 'Point',
           'coordinates' => [
               $row['longitude'],
               $row['latitude'],
           ],
       ],
   ];
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
$.getJSON("http://localhost:8080/geojson-point", function (data) {
  point.addData(data);
  map.addLayer(point);
  // fit map to geojson
  map.fitBounds(point.getBounds());
});
```

## View - Form Input With Coordinate by Marker Location
[https://anshori.github.io/leaflet-search-coordinates/](https://anshori.github.io/leaflet-search-coordinates/)

___    
> [unsorry@2023](https://unsorry.net)