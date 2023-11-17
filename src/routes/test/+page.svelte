<script lang="ts">
    import {onMount} from "svelte";

    class Player{
        public  n: number
        public  x: number
        public  y: number

        public  w: number
        public  h: number

        public  el: HTMLElement

        constructor(num: number, element: HTMLElement){
            this.el = element;
            this.n = num;

            if(num===1){
                this.x = 250;
            }else{
                this.x = 2040;//canvasW = 18888 Pw = 70 Abstand = 250
            }
            this.y = 375;
            this.w = 40;
            this.h = 180;

        }

      refresh(){
        this.el.style.left = this.x+"px";
        this.el.style.top = this.y+"px";
      }

      update(y: number) {
        //this.y = y;
        this.refresh()
      }
    }


    class Ball {
      public x: number
      public y: number

      public w: number
      public h: number

      public el: HTMLElement

      constructor(element: HTMLElement) {

        this.el = element;
        this.x = 1100;
        this.y = 500;
        this.w = 50;
        this.h = 50;

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

    let canvas: undefined | HTMLElement;
    let bel: undefined | HTMLElement;
    let p1el: undefined | HTMLElement;
    let p2el: undefined | HTMLElement;

    // -----

    onMount(() => {
      let p1 = new Player(1, p1el as HTMLElement);
      let p2 = new Player(2, p2el as HTMLElement);
      let ball = new Ball(bel as HTMLElement);

      function refresh(){
        // TODO: Backend Daten holen

        p1.update(0)
        p2.update(0)
        ball.update(0, 0)
      }

      setInterval(function(){
        refresh();
      }, 66.6);
    })

</script>

<main class="w-full min-h-screen justify-center items-center">
  <div class="m-0 p-0 box-border relative top-0 bottom-0 right-0 w-[2360px] h-[1240px] bg-black before:absolute before:z-10 before:top-0 before:bottom-0 before:w-2 before:left-[200px] before:bg-green-500 after:absolute after:z-10 after:top-0 after:bottom-0 after:w-2 after:right-[200px] after:bg-green-500" id="canvas" bind:this={canvas}>
    <div id="p1" class="w-[70px] h-[400px] top-[150px] bg-yellow-300 absolute" bind:this={p1el}>
    </div>
    <div id="p2" class="w-[70px] h-[400px] top-[150px] bg-yellow-300 absolute" bind:this={p2el}>
    </div>
    <div id="ball" class="w-[100px] h-[100px] bg-red-500 rounded-[400px] absolute" bind:this={bel}>
    </div>
  </div>
</main>
