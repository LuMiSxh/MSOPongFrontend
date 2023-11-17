
  <style>
      :root{
          --bg_main: gray;
          --bg_player: blue;
          --bg_title_column: yellow;
          --bg_option: red;
          --bg_option_hover: rgb(186, 5, 5);
          --bg_option_focus: rgb(113, 3, 3);
          --bg_ready: purple;
          --bg_ready_hover: rgb(104, 2, 104);
          --bg_ready_focus: rgb(58, 0, 58);
          --border_ready: rgb(40, 0, 40);
      }
      #playerP{
          width: 1200px;
          height: 100px;
          font-size: 50px;
          display: flex;
          align-items: center;
          justify-content: center;
          text-align: center;
          background-color: var(--bg_player);
          position: relative;
          margin: auto;
      }
      section{
          display: flex;
          height: 600px;
          width: 1400px;
          margin: auto;
          position: relative;
      }
      .column{
          width: 50%;
          height: 100%;
          background: var(--bg_main);
          box-sizing: border-box;
      }
      .column#c1{
          border-right: 1px solid black;
      }
      .column#c2{
          border-left: 1px solid black;
      }
      .column p{
          background: var(--bg_title_column);
          width: 350px;
          height: 100px;
          display: flex;
          align-items: center;
          justify-content: center;
          text-align: center;
          font-size: 40px;
          margin: auto;
          margin-bottom: 40px;
      }
      .option{
          background: var(--bg_option);
          width: 200px;
          height: 60px;
          margin: auto;
          font-size: 30px;
          display: flex;
          justify-content: center;
          align-items: center;
          border: 0;
          text-align: center;
      }
      .option:hover{
          background: var(--bg_option_hover);
      }
      .option:focus{
          background: var(--bg_option_focus);
      }
      #startk{
          position: absolute;
          width: 100px;
          height: 100px;
          border-radius: 500px;
          background-color: var(--bg_ready);
          display: flex;
          align-items: center;
          justify-content: center;
          text-align: center;
          left: 0;
          right: 0;
          top: 0;
          bottom: 0;
          font-size: 25px;
          border: 4px solid var(--border_ready);
          margin: auto;

      }
      #startk:hover{
          background-color: var(--bg_ready_hover);
      }
      #startk:focus{
          background-color: var(--bg_ready_focus);
      }
  </style>
<div id="playerP">Player1</div>
<section>
  <div class="column" id="c1">
    <p>Geschwindigkeit der Ball</p>
    <button class="option gesch">Slow</button>
    <button class="option gesch">Medium</button>
    <button class="option gesch">High</button>
    <button class="option gesch">Crazy</button>
    <button class="option gesch">Imposible</button>
  </div>
  <div class="column" id="c2">
    <p>Grosse der Ball</p>
    <button class="option grosse">Small</button>
    <button class="option grosse">Medium</button>
    <button class="option grosse">Big</button>
    <button class="option grosse">XXXXL</button>
    <button class="option grosse">Dinosaur</button>
  </div>

  <button id="startk">Bereit</button>
</section>

<script>
  let geschAktiv = null;
  let grosseAktiv = null;
  let geschOptions = document.querySelectorAll(".gesch");
  let grosseOptions = document.querySelectorAll(".grosse");

  geschOptions.forEach((option, idx)=>{
    option.addEventListener("click", function(){
      geschAktiv = idx;
    })
  })
  grosseOptions.forEach((option, idx)=>{
    option.addEventListener("click", function(){
      grosseAktiv = idx;
    })
  })


  document.getElementById("readyk").addEventListener("click", function(){
    let daten = {GE:geschAktiv, GR: grosseAktiv};
    //server aufruf
  })
</script>
