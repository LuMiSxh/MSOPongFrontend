<script lang="ts">
    import {onMount} from "svelte";
    import {browser} from "$app/environment";
    import { PUBLIC_WEBSOCKET } from "$env/static/public";

    let ws: null | WebSocket = null;
    let gameStarted = false;
    let playerReady = false;
    let winner = 0;
    let player = 1;

    class Player{
        public  y: number
        public  el: HTMLElement

        constructor(element: HTMLElement, y: number){
            this.el = element;
            this.y = y;
        }

        moveNatural(y: number) {
            if (!y) return
            this.y = this.y + y;
            this.el.style.top = this.y + "px"
        }

        refresh(){
            this.el.style.top = this.y+"px";
        }

        update(y: number) {
            this.y = y;
            this.refresh()
        }
    }


    class Ball {
        public x: number
        public y: number
        public el: HTMLElement

        constructor(element: HTMLElement, x: number, y: number) {

            this.el = element;
            this.x = x;
            this.y = y;
        }

        refresh() {
            this.el.style.left = this.x + "px";
            this.el.style.top = this.y + "px";
        }

        update(x: number, y: number) {
            this.x = x;
            this.y = y;
            this.refresh()
        }
    }

    // Elements
    let canvas: undefined | HTMLElement;
    let bel: undefined | HTMLElement;
    let p1el: undefined | HTMLElement;
    let p2el: undefined | HTMLElement;

    // CONSTS //
    // Canvas
    const canvasWidth = 1420; //
    const canvasHeight = 580; //
    const canvasColor = "#000000"
    const canvasImage = "/Logo.png";  // default: null
    const canvasLineWidth = 1;
    const canvasLinePadding = 50; //
    const canvasLineColor = "#3f12e3"
    // Paddles
    const paddleRate = 10; //
    const paddleRateMax = 100;
    const paddleRateMultiplier = 0.4;
    const paddleHeight = 200;
    const paddleWidth = 15; //
    const paddlePadding = 60;
    const paddleColor = "#FFFFFF"
    // Ball
    const ballRadius = 40;
    const ballColor = "#574288"
    // Movement calculations
    const movementMax = canvasHeight - paddleHeight; //
    const middleY = canvasHeight / 2; //

    // -----

    onMount(() => {
        ws = new WebSocket(PUBLIC_WEBSOCKET);
        ws.onopen = () => {
            ws?.send(JSON.stringify({
                "Protocol": "Reset"
            }));
        }

        let yourself = new Player(
          player === 1? p1el as HTMLElement : p2el as HTMLElement,
          (canvasHeight / 2) - (paddleHeight / 2)
        );
        let opponent = new Player(
          player === 2? p1el as HTMLElement : p2el as HTMLElement,
          (canvasHeight / 2) - (paddleHeight / 2)
        );
        let ball = new Ball(
          bel as HTMLElement,
          (canvasWidth / 2) - (ballRadius),
          (canvasHeight / 2) - (paddleHeight / 2)
        );

        let move = false;
        let moveY = 0;
        let moveRuns = 0;

        // Event loop
        function refresh(){
            // Move Paddle
            const clamp = (num: number, min: number, max: number): number => Math.min(Math.max(num, min), max);
            if (move) {
                moveRuns++;
                if (moveY > middleY) {
                    if ((yourself.y + clamp((paddleRate * (paddleRateMultiplier * moveRuns)), 0, paddleRateMax)) < movementMax) {
                        yourself.moveNatural(clamp((paddleRate * (paddleRateMultiplier * moveRuns)), 0, paddleRateMax));
                    } else {
                        yourself.update(movementMax)
                        moveRuns = 0;
                    }
                } else {
                    if ((yourself.y - clamp((paddleRate * (paddleRateMultiplier * moveRuns)), 0, paddleRateMax)) > 0) {
                        yourself.moveNatural(-clamp((paddleRate * (paddleRateMultiplier * moveRuns)), 0, paddleRateMax));
                    } else {
                        yourself.update(0)
                        moveRuns = 0;
                    }
                }
            }

            // Send data to server
            let gameObj = {
                "Protocol": "GameData",
                "CurrentPlayer": player,
                "1XPos": player === 1? p1el?.offsetLeft : p2el?.offsetLeft,
                "1YPos": player === 1? p1el?.offsetTop : p2el?.offsetLeft,
                "2XPos": player === 2? p2el?.offsetLeft : p1el?.offsetLeft,
                "2YPos": player === 2? p2el?.offsetTop : p1el?.offsetLeft,
                "PaddleHeight": paddleHeight,
                "BallXPos": bel?.offsetLeft,
                "BallYPos": bel?.offsetTop,
                "BallRadius": ballRadius,
                "CanvasWidth": canvasWidth,
                "CanvasHeight": canvasHeight
            }

            ws?.send(JSON.stringify(gameObj));
        }

        // Websocket
        ws.onmessage = (ev) => {
            let wsdata = JSON.parse(ev.data)
            switch (wsdata.types) {
                case "GameStart":
                    gameStarted = true;
                    yourself.update(player === 1? wsdata.P1Y : wsdata.P2Y)
                    opponent.update(player === 2? wsdata.P1Y : wsdata.P2Y)
                    ball.update(wsdata.BX, wsdata.BY)
                    break;
                case "GameData":
                    yourself.update(player === 1? wsdata.P1Y : wsdata.P2Y)
                    opponent.update(player === 2? wsdata.P1Y : wsdata.P2Y)
                    ball.update(wsdata.BX, wsdata.BY)
                    break;
                case "GameEnd":
                    winner = wsdata.CurrentPlayer;
                    gameStarted = false;
                    playerReady = false;
                    break;
                case "GameReset":
                    winner = 0;
                    gameStarted = false;
                    playerReady = false;
                    break;
            }
        }

        setInterval(function(){
            refresh();
        }, 66.6);

        // Event listeners
        if (browser) {
            canvas?.addEventListener("mousedown", function (event) {
                moveY = event.pageY;
                move = true;
            });
            canvas?.addEventListener("mousemove", function (event) {
                if (move) moveY = event.pageY;
            });
            canvas?.addEventListener("mouseup", function () {
                move = false;
                moveRuns = 0;
            });
            canvas?.addEventListener('touchstart', function(event) {
                moveY = event.touches[0].pageY;
                move = true;
            });
            canvas?.addEventListener("touchmove", function (event) {
                if (move) moveY = event.touches[0].pageY;
            });
            canvas?.addEventListener('touchend', function() {
                move = false;
                moveRuns = 0;
            });
        }
    });

    function startGame() {
        playerReady = true;
    }
</script>

<main class="w-full h-screen overflow-hidden bg-red-600 relative">
    <div
      class="p-0 box-border absolute top-0 right-0 bg-no-repeat bg-contain bg-center"
      style="width: {canvasWidth}px; height: {canvasHeight}px; background-color: {canvasColor}; background-image: url('{canvasImage ?? 'none'}');"
      id="canvas"
      bind:this={canvas}
    >
        <!-- Separator Lines --->
        <div
          class="absolute h-full"
          style="background-color: {canvasLineColor}; width: {canvasLineWidth}px; left: {canvasLinePadding}px;"
        >
        </div>
        <div
          class="absolute h-full"
          style="background-color: {canvasLineColor}; width: {canvasLineWidth}px; left: {canvasWidth - canvasLinePadding - canvasLineWidth}px;"
        >
        </div>
        <!-- Scoreboard --->
        <h1
          class="absolute text-9xl font-bold text-stone-600 w-[170px] text-center z-10 select-none hidden"
          style="left: {(canvasWidth / 2) - 85}px; top: 10px;"
        >
            00
        </h1>
        <!-- Paddles --->
        <div
          id="p1"
          class="absolute z-20"
          bind:this={p1el}
          style="height: {paddleHeight}px; width: {paddleWidth}px; background-color: {paddleColor}; left: {paddlePadding}px; top: {(canvasHeight / 2) - (paddleHeight / 2)}px;"
        >
        </div>
        <div
          id="p2"
          class="absolute z-20"
          bind:this={p2el}
          style="height: {paddleHeight}px; width: {paddleWidth}px; background-color: {paddleColor}; left: {canvasWidth - paddlePadding - paddleWidth}px; top: {(canvasHeight / 2) - (paddleHeight / 2)}px;"
        >
        </div>
        <!-- Ball --->
        <div
          id="ball"
          class="absolute aspect-square z-20 border-2"
          style="width: {ballRadius * 2}px; background-color: {ballColor}; top: {(canvasHeight / 2) - ballRadius}px; left: {(canvasWidth / 2) - ballRadius}px; border-color: {canvasColor};"
          bind:this={bel}
        >
        </div>
    </div>
    {#if gameStarted === false}
        <div
          class="p-0 box-border absolute top-0 right-0 bg-no-repeat bg-contain bg-center"
          style="width: {canvasWidth}px; height: {canvasHeight}px; background: {canvasColor}80;"
        >
            {#if playerReady === false}
                {#if winner !== 0}
                    <div
                      class="absolute z-50 text-4xl w-[150px] h-[150px] bg-stone-600 rounded-full"
                      style="top: {(canvasHeight / 2) - 75}px; left: {(canvasWidth / 2) - 75}px;"
                    >
                        Gewonnen hat: Spieler {winner}
                    </div>
                {/if}
                <button
                  class="absolute z-50 text-4xl w-[150px] h-[150px] bg-stone-600 rounded-full"
                  style="top: {(canvasHeight / 2) - 75}px; left: {(canvasWidth / 2) - 75}px;"
                  on:click={startGame}
                  on:focus
                >
                    Bereit?
                </button>
            {:else}
                <div>
                </div>
            {/if}
        </div>
    {/if}
</main>

