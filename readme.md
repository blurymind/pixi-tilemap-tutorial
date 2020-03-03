# About

Pixi-tilemap allows a developer to create tilemaps based on PIXI.js and https://github.com/pixijs/pixi-tilemap

# Example Usage

Completed demo: https://blurymind.github.io/pixi-tilemap-tutorial/

## Creating a basic tile map

```javascript
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pixi.js/4.6.2/pixi.min.js"></script>
    <script src="./lib/pixi-tilemap.js"></script>
    <script src="//rawgit.com/mrdoob/stats.js/master/build/stats.min.js"></script>
  </head>
  <body>
    <script>
      // We create a class where to store the sliced tiles from the tileset image resource
      class TileSet {
        constructor({
          tilewidth,
          tileheight,
          texture,
          offset,
          count,
          scaleMode
        }) {
          this.tilewidth = tilewidth;
          this.tileheight = tileheight;
          this.offset = offset || 0;
          this.texture = texture;
          this.textureCache = [];
          this.scaleMode = scaleMode || PIXI.SCALE_MODES.NEAREST;
          this.prepareTextures(count);
        }
        get width() {
          return this.texture.width;
        }
        get height() {
          return this.texture.height;
        }
        prepareTextures(count) {
          var size =
            count ||
            (this.width / this.tilewidth) * (this.height / this.tileheight);

          this.textureCache = new Array(size)
            .fill(0)
            .map((_, frame) => this.prepareTexture(frame));
        }
        prepareTexture(frame) {
          var cols = Math.floor(this.width / this.tilewidth);
          var x = ((frame - this.offset) % cols) * this.tilewidth;
          var y = Math.floor((frame - this.offset) / cols) * this.tileheight;
          var rect = new PIXI.Rectangle(x, y, this.tilewidth, this.tileheight);
          var texture = new PIXI.Texture(this.texture, rect);

          texture.baseTexture.scaleMode = this.scaleMode;
          texture.cacheAsBitmap = true;

          return texture;
        }
        getFrame(frame) {
          if (!this.textureCache[frame]) {
            this.prepareTexture(frame);
          }

          return this.textureCache[frame];
        }
      }
      var TIME = 0;
      var resolutionX = 1024;
      var resolutionY = 720;

      var stats = new Stats();
      stats.showPanel(0);
      document.body.appendChild(stats.dom);

      var app = new PIXI.Application(resolutionX, resolutionY);
      document.body.appendChild(app.view);

      // We need to load the tileset image resource and the exported json file from tiled that stores the tilemap and tileset data
      PIXI.loader.add(["imgs/Viking3.png", "imgs/island.json"]).load(setup);

      function setup(loader, resources) {
        PIXI.tilemap.Constant.boundSize = 2048;
        PIXI.tilemap.Constant.bufferSize = 4096;
        console.log(loader, resources);
        var island = resources["imgs/island.json"].data;

        // Here we take the first tileset in the tiled file. If you have multiple, you might need to do something different
        var tileset = island.tilesets[0];
        var { tileheight, tilewidth, tilecount } = tileset;

        var TILESET = new TileSet({
          tilewidth,
          tileheight,
          texture: PIXI.utils.TextureCache["imgs/Viking3.png"],
          offset: 1,
          count: tilecount,
          tileset,
          scaleMode: PIXI.SCALE_MODES.NEAREST
        });

        var TILEMAP = new PIXI.tilemap.CompositeRectTileLayer(0);
        app.stage.addChild(TILEMAP);

        island.layers.forEach(layer => {
          if (!layer.visible) return;
          if (layer.type === "objectgroup") {
            layer.objects.forEach(object => {
              var { gid, id, width, height, x, y, visible } = object;
              if (visible === false) return;
              if (TILESET.getFrame(gid)) {
                TILEMAP.addFrame(TILESET.getFrame(gid), x, y - tileheight);
              }
            });
          } else if (layer.type === "tilelayer") {
            var ind = 0;
            for (var i = 0; i < layer.height; i++) {
              for (var j = 0; j < layer.width; j++) {
                var xPos = tilewidth * j;
                var yPos = tileheight * i;

                var tileUid = layer.data[ind];

                if (tileUid !== 0) {
                  var tileData = tileset.tiles.find(
                    tile => tile.id === tileUid - 1
                  );

                  // Animated tiles have a limitation with only being able to use frames arranged one to each other on the image resource
                  if (tileData && tileData.animation) {
                    TILEMAP.addFrame(
                      TILESET.getFrame(tileUid),
                      xPos,
                      yPos
                    ).tileAnimX(tilewidth, tileData.animation.length);
                  } else {
                    // Non animated props dont require tileAnimX or Y
                    TILEMAP.addFrame(TILESET.getFrame(tileUid), xPos, yPos);
                  }
                }

                ind += 1;
              }
            }
            app.start();
          }
        });
        gameLoop();
      }

      // For tracking stats
      function gameLoop() {
        stats.begin();
        stats.end();
        requestAnimationFrame(gameLoop);
      }

      // We increment time by one whenever we want to progress all tile animations
      setInterval(() => {
        TIME += 1;
        // tileAnim[0] will move time on the X axis, while tileAnim[1] on the Y
        app.renderer.plugins.tilemap.tileAnim[0] = TIME;
        app.renderer.render(app.stage);
      }, 100);
    </script>
  </body>
</html>
```
