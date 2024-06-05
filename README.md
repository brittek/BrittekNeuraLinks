"NeuraLinks by Brittek"

This project visualizes the connection of nodes in a 3D space, generating a dynamic and interactive display of connecting lines and data points. The animation uses HTML5 canvas and JavaScript to render the visual effects.
Features

    Dynamic Node Connections: Nodes connect and form structures dynamically based on predefined parameters.
    Interactive Controls: Adjust the direction, blur, and parallax effects using the controls provided.
    Responsive Design: The canvas resizes according to the window size, ensuring a consistent experience across devices.
    Aesthetic Design: Smooth animations and a dark-themed background create a visually appealing experience.


Files

    index.html: The main HTML file that sets up the canvas and includes the controls.
    index.css: The CSS file that styles the canvas and the controls.
    index.js: The JavaScript file that contains the logic for node connections, animations, and controls.

Code Overview
index.js

    Initialization:

    js

var w = (c.width = window.innerWidth),
    h = (c.height = window.innerHeight),
    ctx = c.getContext("2d"),
    opts = { /* options */ },
    connections = [],
    toDevelop = [],
    data = [],
    all = [],
    tick = 0,
    animating = false,
    Tau = Math.PI * 2;

ctx.fillStyle = "#222";
ctx.fillRect(0, 0, w, h);
ctx.fillStyle = "#ccc";
ctx.font = "50px Verdana";
ctx.fillText("Calculating Nodes", w / 2 - ctx.measureText("Calculating Nodes").width / 2, h / 2 - 15);

window.setTimeout(init, 4);

Connection Class:

js

function Connection(x, y, z, size) {
    this.x = x;
    this.y = y;
    this.z = z;
    this.size = size;
    this.screen = {};
    this.links = [];
    this.isEnd = false;
    this.glowSpeed = opts.baseGlowSpeed + opts.addedGlowSpeed * Math.random();
}

Connection.prototype.link = function () { /* linking logic */ };
Connection.prototype.step = function () { /* stepping logic */ };
Connection.prototype.draw = function () { /* drawing logic */ };

Data Class:

js

function Data(connection) {
    this.glowSpeed = opts.baseGlowSpeed + opts.addedGlowSpeed * Math.random();
    this.speed = opts.baseSpeed + opts.addedSpeed * Math.random();
    this.screen = {};
    this.setConnection(connection);
}

Data.prototype.step = function () { /* data stepping logic */ };
Data.prototype.draw = function () { /* data drawing logic */ };
Data.prototype.setConnection = function (connection) { /* connection setting logic */ };

Animation Function:

js

    function anim() {
        window.requestAnimationFrame(anim);
        ctx.globalCompositeOperation = "source-over";
        ctx.fillStyle = opts.repaintColor;
        ctx.fillRect(0, 0, w, h);

        ++tick;
        var rotX = tick * opts.rotVelX,
            rotY = tick * opts.rotVelY;

        cosX = Math.cos(rotX);
        sinX = Math.sin(rotX);
        cosY = Math.cos(rotY);
        sinY = Math.sin(rotY);

        if (data.length < connections.length * opts.dataToConnections) {
            var datum = new Data(connections[0]);
            data.push(datum);
            all.push(datum);
        }

        ctx.globalCompositeOperation = "lighter";
        ctx.beginPath();
        ctx.lineWidth = opts.wireframeWidth;
        ctx.strokeStyle = opts.wireframeColor;
        all.forEach(item => item.step());
        ctx.stroke();
        ctx.globalCompositeOperation = "source-over";
        all.sort((a, b) => b.screen.z - a.screen.z);
        all.forEach(item => item.draw());
    }

index.css

    Canvas Styling:

    css

canvas {
    position: absolute;
    top: 0;
    left: 0;
}

Body Styling:

css

body {
    display: grid;
    place-items: center;
    min-height: 100vh;
    font-family: "SF Pro Text", "Helvetica Neue", Helvetica, Arial, sans-serif;
    background-size: cover;
    background-position: center bottom;
}

body::before {
    content: "";
    position: fixed;
    inset: 0;
    background: hsl(0 0% 0% / 0.75);
    z-index: -1;
}

Controls Styling:

css

    .controls {
        position: fixed;
        top: 1rem;
        right: 1rem;
        display: grid;
        grid-template-columns: auto 1fr;
        gap: 0.5rem 1rem;
        color: white;
        font-weight: 300;
        align-items: center;
    }

Deploying to GitHub Pages

To deploy this project to GitHub Pages using GitHub Actions, follow these steps:

    Create GitHub Workflow: Create a .github/workflows/deploy.yml file with the following content:

    yaml

    name: Deploy Next.js site to Pages

    on:
      push:
        branches: ["main"]
      workflow_dispatch:

    permissions:
      contents: read
      pages: write
      id-token: write

    concurrency:
      group: "pages"
      cancel-in-progress: false

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout
            uses: actions/checkout@v4
          - name: Detect package manager
            id: detect-package-manager
            run: |
              if [ -f "${{ github.workspace }}/yarn.lock" ]; then
                echo "manager=yarn" >> $GITHUB_OUTPUT
                echo "command=install" >> $GITHUB_OUTPUT
                echo "runner=yarn" >> $GITHUB_OUTPUT
                exit 0
              elif [ -f "${{ github.workspace }}/package.json" ]; then
                echo "manager=npm" >> $GITHUB_OUTPUT
                echo "command=ci" >> $GITHUB_OUTPUT
                echo "runner=npx --no-install" >> $GITHUB_OUTPUT
                exit 0
              else
                echo "Unable to determine package manager"
                exit 1
              fi
          - name: Setup Node
            uses: actions/setup-node@v4
            with:
              node-version: "20"
              cache: ${{ steps.detect-package-manager.outputs.manager }}
          - name: Setup Pages
            uses: actions/configure-pages@v5
            with:
              static_site_generator: next
          - name: Restore cache
            uses: actions/cache@v4
            with:
              path: |
                .next/cache
              key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json', '**/yarn.lock') }}-${{ hashFiles('**.[jt]s', '**.[jt]sx') }}
              restore-keys: |
                ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json', '**/yarn.lock') }}-
          - name: Install dependencies
            run: ${{ steps.detect-package-manager.outputs.manager }} ${{ steps.detect-package-manager.outputs.command }}
          - name: Build with Next.js
            run: ${{ steps.detect-package-manager.outputs.runner }} next build
          - name: Upload artifact
            uses: actions/upload-pages-artifact@v3
            with:
              path: ./out

      deploy:
        environment:
          name: github-pages
          url: ${{ steps.deployment.outputs.page_url }}
        runs-on: ubuntu-latest
        needs: build
        steps:
          - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v4

    Commit and Push: Commit the changes to your repository and push to the main branch.

    Deploy: The GitHub Actions workflow will automatically build and deploy your Next.js site to GitHub Pages.

Contributing

Contributions are welcome! Please fork the repository and create a pull request with your changes.
License

This project is licensed under the MIT License.
Contact

For more information, feel free to reach out:

    Website: https://brittek.digital
    Instagram: @brittekdgtl
    LinkedIn: James Britton
    GitHub: @brittek
    PayPal: Paypal.Me/BrittekDigtl
