# Node Gacha

Node Gacha (named after the Japanese onomatopoeia for the sound made by toy machines) aims to be a module providing different methods for calculating randomness in games.
Randomness can include everything from enemy encounters to item drop rates, particularly useful for RPG's and Roguelikes.

Eventually we'll offer a few different systems, but for now we only support "roguelike".

```
npm install gacha
```

## Roguelike

**Purpose:** Given a listing of items (or enemies) with variable drop rates, determine the odds of them being dropped on a particular level.

### Diagram

![Diagram](https://github.com/tlhunter/node-gacha/blob/master/screenshot.png)

The above diagram gives a visualization of how Roguelike works.
Each item has an ideal level (center of curve), a spread (the radius of the curve), and a weight (makes it taller).
After passing data through the roguelike function, a list of probabilities of each item spawning is provided on a per-level basis.

### Code Sample

```javascript
var gacha = require('gacha');

// items must be an object where each property contains level, weight, and spread.
var items = {
  "101": {
    "name": "Knife",
    "type": "weapon",
    "level": 1,
    "weight": 1.0,
    "spread": 1
  },
  "102": {
    "name": "Sword",
    "type": "weapon",
    "level": 2,
    "weight": 1.0,
    "spread": 2
  },
  "103": {
    "name": "Iron Buckler",
    "type": "shield",
    "level": 3,
    "weight": 3.0,
    "spread": 3
  },
  "104": {
    "name": "Potion",
    "type": "heal",
    "level": 3,
    "weight": 1.0,
    "spread": 2
  }
};

var equip = gacha.roguelike(items, function(item) {
  // Optional filter callback, defaults to `return true;`
  return item.type !== 'heal';
});

console.log(equip); // Shown below

// Which item should we spawn on level 3?
var lvl3 = equip[3];
var strata = Math.random() * lvl3.total;
var item = null;

for (var i = 0; i < lvl3.strata.length; i++) {
  if (strata <= lvl3.strata[i]) {
    var id = lvl3.lookup[i];
    item = items[id];
    break;
  }
}

console.log("Receiving item for level three:", item);
```

### Resulting `equip` object

```json
{
  "0": {
    "lookup": [ "101", "102", "103" ],
    "strata": [ 0.24197072451914337, 0.3457475988742921, 0.4485344854101383 ],
    "total": 0.4485344854101383
  },
  "1": {
    "lookup": [ "101", "102", "103" ],
    "strata": [ 0.3989422804014327, 0.6186379251352939, 0.8551480729542124 ],
    "total": 0.8551480729542124
  },
  "2": {
    "lookup": [ "101", "102", "103" ],
    "strata": [ 0.24197072451914337, 0.5240655162930214, 0.9140048277385038 ],
    "total": 0.9140048277385038
  },
  "3": {
    "lookup": [ "102", "103" ],
    "strata": [ 0.2196956447338612, 0.6803545106956419 ],
    "total": 0.6803545106956419
  },
  "4": {
    "lookup": [ "102", "103" ],
    "strata": [ 0.10377687435514871, 0.493716185800631 ],
    "total": 0.493716185800631
  },
  "5": {
    "lookup": [ "103" ],
    "strata": [ 0.2365101478189184 ],
    "total": 0.2365101478189184
  },
  "6": {
    "lookup": [ "103" ],
    "strata": [ 0.1027868865358462 ],
    "total": 0.1027868865358462
  }
}
```
