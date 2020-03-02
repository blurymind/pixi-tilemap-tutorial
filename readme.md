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
          const size =
            count ||
            (this.width / this.tilewidth) * (this.height / this.tileheight);

          this.textureCache = new Array(size)
            .fill(0)
            .map((_, frame) => this.prepareTexture(frame));
        }
        prepareTexture(frame) {
          const cols = Math.floor(this.width / this.tilewidth);
          const x = ((frame - this.offset) % cols) * this.tilewidth;
          const y = Math.floor((frame - this.offset) / cols) * this.tileheight;
          const rect = new PIXI.Rectangle(
            x,
            y,
            this.tilewidth,
            this.tileheight
          );
          const texture = new PIXI.Texture(this.texture, rect);

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
      var resolutionX = 800;
      var resolutionY = 600;
      var tileSizeX = 128;
      var tileSizeY = 128;

      var stats = new Stats();
      stats.showPanel(0);
      document.body.appendChild(stats.dom);

      var app = new PIXI.Application(resolutionX, resolutionY);
      document.body.appendChild(app.view);

      var groundTiles;
      var playerTankSprite;
      var playerOffsetX = resolutionX / 2 - 24;
      var playerOffsetY = resolutionY / 2 - 24;

      var player = {
        x: 0,
        y: 0
      };

      PIXI.loader.add(["imgs/Viking3.png", "imgs/island.json"]).load(setup);

      function setup(loader, resources) {
        PIXI.tilemap.Constant.boundSize = 2048;
        PIXI.tilemap.Constant.bufferSize = 4096;
        console.log(loader, resources);
        var island = resources["imgs/island.json"].data;

        let tileset = island.tilesets[0];
        const { tileheight, tilewidth, tilecount } = tileset;

        const TILESET = new TileSet({
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
          console.log("LAYER>>", layer);
          if (layer.type === "objectgroup") {
            layer.objects.forEach(object => {
              const { gid, id, width, height, x, y, visible } = object;
              if (visible === false) return;
              if (TILESET.getFrame(gid)) {
                TILEMAP.addFrame(TILESET.getFrame(gid), x, y - tileheight);
              }
            });
          } else if (layer.type === "tilelayer") {
            let ind = 0;
            for (var i = 0; i < layer.height; i++) {
              for (var j = 0; j < layer.width; j++) {
                const xPos = tilewidth * j;
                const yPos = tileheight * i;

                const tileUid = layer.data[ind];

                if (tileUid !== 0) {
                  const tileData = tileset.tiles.find(
                    tile => tile.id === tileUid - 1
                  );

                  if (tileData && tileData.animation) {
                    TILEMAP.addFrame(
                      TILESET.getFrame(tileUid),
                      xPos,
                      yPos,
                      1,
                      0,
                      tileData.animation.length * tilewidth
                    );
                  } else {
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

      var tick = new Date().getTime();
      var tileAnim = 0;
      var tileAnimationTick = 0;
      function gameLoop() {
        stats.begin();
        stats.end();
        requestAnimationFrame(gameLoop);
      }

      setInterval(() => {
        TIME += 42;
        app.renderer.plugins.tilemap.tileAnim[0] = TIME;
        app.renderer.render(app.stage);
      }, 100);
    </script>
  </body>
</html>
```
