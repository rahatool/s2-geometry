# Geometry on the Sphere - JavaScript Library
A native JavaScript port of Google's S2 Geometry library for dealing with spatial data.

---
The S2 Geometry is based on projecting the earth sphere onto a cube, with some scaling of face coordinates to keep things close to approximate equal area for adjacent cells.

To convert lat,lng to a cell id:
- convert lat,lng to x,y,z
- convert x,y,z to face,u,v
- u,v scaled to s,t with quadratic formula
- s,t converted to integer i,j offsets
- i,j converted to a position along a Hilbert space-filling curve
- combine face,position to get the cell id

![Hierarchy of S2 Geometry](https://s2geometry.io/devguide/img/s2hierarchy.gif)

# Installation
## CDN
Import library from the ESM.SH CDN for fast and easy setup:
### In web browser
Simply include s2-geometry in your html `<header>` tag.
```html
<script type="module">
import {S2LatLng, S2Cell} from "//esm.sh/gh/rahatool/s2-geometry";
</script>
```
### In web worker
Some s2-geometry functions that do not access nor modify the DOM can be used in web workers.

In order to be able to use s2-geometry in those web workers, you need to import the source file as an ES6 module using:
```javascript
import {S2LatLng, S2Cell} from "//esm.sh/gh/rahatool/s2-geometry";
```
## NPM registry
Use the package manager npm to install s2-geometry.
```shell
$ npm install github:rahatool/s2-geometry
```
Library will be copied to `node_modules/@raha.group/s2-geometry` directory.
## Direct Download
Grab the [latest release](/archive/refs/heads/development.zip) file.

# Implemented APIs

## S2LatLng API

### Skeleton of S2LatLng
```javascript
S2LatLng := ObjectModel {
	@Static S2LatLng from(Number latitude, Number longitude);
	@Static S2LatLng fromInteger(BigInt identifier);
	BigInt toInteger(Level level = MAX_LEVEL);
	SequenceOf(S2Cell) getNeighbors(level);
	BigInt compareTo(S2LatLng other);
};
Level := RangeLimit<Number> {
	lower := 1,
	upper := 30
};
MAX_LEVEL := 30;
```

### Usage of S2LatLng
```javascript
let latitude = 40.2574448;
let longitude = -111.7089464;
// Main factory method to create a S2LatLng object from pair (latitude, longitude)
let point = S2LatLng.from(latitude, longitude);

let level = 15; // Range of level is between 1 and 30
let cellId = point.toInteger(level);
console.log(cellId);
// It prints 9749618446378729472n

// Factory method to create a S2LatLng object from BigInt
point = S2LatLng.fromInteger(cellId);
console.log(point);
// It prints {lat: 40.2574448, lng: -111.7089464}

for (let cell of point.getNeighbors(level)) {
	console.log(cell);
}
```

## S2Cell API
Currently contains basic support for S2Cell.

<table>
	<caption>Map of Faces</caption>
	<tr><td></td><td>Face: 2<br />Orientation: A<br />Cover: Africa</td><td colspan="2"></td></tr>
	<tr><td>Face: 0<br />Orientation: A<br />Cover: Africa</td><td>Face: 1<br />Orientation: D<br />Cover: Asia</td><td>Face: 3<br />Orientation: D<br />Cover: Australia</td><td>Face: 4<br />Orientation: A<br />Cover: Americas, Provo, UT</td></tr>
	<tr><td colspan="3"></td><td>Face: 5<br />Orientation: D<br />Cover: Antarctica</td></tr>
</table>

The S2 hierarchy is useful for spatial indexing and for approximating regions as a collection of cells. Cells can be used to represent both points and regions: points are generally represented as leaf cells, while regions are represented as collections of cells at any level(s).

Each cell is uniquely identified by a 64-bit S2CellId. The S2 cells are numbered in a special way in order to maximize locality of reference when they are used for spatial indexing (compared to other methods of numbering the cells).

S2CellId combine face and the hilbert curve position into a single 64 bit integer. this gives efficient space and speed.

### Skeleton of S2Cell
```javascript
S2Cell := ObjectModel {
	@Static from(Face face, OrderedPair ij, Level level = MAX_LEVEL); // Internal use
	
	@Static S2Cell fromLatLng(S2LatLng latLng, Level level = MAX_LEVEL);
	S2LatLng toLatLng();

	@Static S2Cell fromInteger(BigInt identifier);
	BigInt toInteger();
	
	SequenceOf(S2LatLng) getCornerLatLngs();
	SequenceOf(S2Cell) getNeighbors();
	S2Cell move(Number step);

	Boolean includes(S2LatLng point);
	Boolean includes(S2Cell cell);
	BigInt compareTo(S2Cell other);
};
Face := RangeLimit<Number> {
	lower := 0,
	upper := 5
};
```

### Usage of S2Cell
```javascript
// Factory method to create a S2Cell object from S2LatLng and level
let cell = S2Cell.fromLatLng(point, level);
console.log(cell.toLatLng().compareTo(point) == 0n);
// It prints true

// Factory method to create a S2Cell object from BigInt
cell = S2Cell.fromInteger(cellId);
console.log(cell.toInteger() == point.toInteger());
// It prints true

for (let cell of point.getNeighbors()) {
	console.log(cell);
}
// It prints the neighbors from left, down, right and up.

// You can get the previous and next S2Cell from any given cell:
let nextCell = cell.move(1);
let previousCell = cell.move(-1);

// Does cell include other item?
console.log(cell.includes(point));
```

# Supported platforms
Raha S2-geometry has been tested and works on the following engines:
- Node.js 10.4+
- Chrome (desktop & Android) 67+
- Firefox 68+
- Safari 14+ (desktop & iOS)
- Latest version of other web-browsers

# License
S2-geometry by [Raha Group](//raha.group) is licensed under the [CC-BY-SA-4.0 License](//creativecommons.org/licenses/by-sa/4.0/).

# Resources
- [S2 Geometry Library](//s2geometry.io/)
- [S2Cell Hierarchy](//s2geometry.io/devguide/s2cell_hierarchy)
- [Presentation of Google's S2 Library](//docs.google.com/presentation/d/1Hl4KapfAENAOf4gv-pSngKwvS_jwNVHRPZTTDzXXn6Q)
- [Spatial indexing with Quadtrees and Hilbert Curves](//blog.notdot.net/2009/11/Damn-Cool-Algorithms-Spatial-indexing-with-Quadtrees-and-Hilbert-Curves)

# Alternative projects
- [coolaj86 / s2-geometry.js](//git.coolaj86.com/coolaj86/s2-geometry.js)
- [Node S2Geometry Typescript](//github.com/vekexasia/nodes2-ts/)

# Stay connected
Stay in touch with Raha’s community and keep track of development and community news by subscribing to the Raha’s [Blog](//raha.group) and [YouTube channel](//youtube.com/channel/UC0bWOBL-vMPCwT6nI9Cz1yw).

## Documentation
The official docs are a great place to discover new things.

## Issue Tracker
Are you experiencing problems with Raha? [report issues](//groups.google.com/d/forum/rahagroup) and find solutions to your problems here. 

## Feature Request
You are always welcome to ask for more features to be added to Raha.

## Events
Stay up to date with meetups, conferences and more.

## Support
Looking for help? please first check out the official (mentioned above) and unofficial (e.g. [Stack Overflow](//stackoverflow.com/questions/tagged/raha)) resources. If you are still experiencing problems, feel free to create a new issue.

# Donation
If you'd like Raha to grow even stronger, please become a sponsor today by donating via Paypal [![Donate][paypal-image]][paypal-url] *(Farnood)* to support Raha's ongoing maintenance and development of new functionality.

[paypal-image]: https://img.shields.io/badge/paypal-donate-brightgreen.svg
[paypal-url]: //paypal.com/cgi-bin/webscr?cmd=_donations&business=RZC8HCR5SPGQY&item_name=Contribute+to+Raha%27s+ongoing+maintenance+and+development&currency_code=USD&source=url