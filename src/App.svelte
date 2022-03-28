<script>
  import pnp from './assets/picknplace.json'
  
  const pcbWidth = 40.3
  let selectedComponent = {}
  let components = pnp
  let widthMultiplier
  const types = {
    c: 'Capacitor',
    l: 'Inductor',
    r: 'Resistor',
    u: 'IC',
    q: 'Transistor',
    d: 'Diod',
    x: 'Crystal'
  }
  let pcb = null

  const selectComponent = (e) => {
    const selected = e.currentTarget.value
    selectedComponent = pnp.filter(c => c.Designator == selected)[0]
  }

  const filterComponents = (f) => {
    if (f === 'r') {
      components = pnp.filter(c => c.Designator.charAt(0).toLowerCase() == 'r')
    }
    if (f === 'c') {
      components = pnp.filter(c => c.Designator.charAt(0).toLowerCase() == 'c')
    }
    if (f === 'ic') {
      components = pnp.filter(c => ['q', 'u'].includes(c.Designator.charAt(0).toLowerCase()))
    }
    if (f === 'all') {
      components = pnp
    }
    if (f === 'others') {
      components = pnp.filter(c => !['c', 'r', 'q', 'u'].includes(c.Designator.charAt(0).toLowerCase()))
    }
  }

  const getType = (t) => {
    if (t.toLowerCase().match(/led/)) {
      return 'LED'
    }
    if (t.toLowerCase().match(/usb/)) {
      return 'USB port'
    }
    return types[t.charAt(0).toLowerCase()] || ''
  }

  const selectFile = e => {
    e.preventDefault()
    document.getElementById('fileElem').click()
  }
  
  
  function handleFiles() {
    if (this.files.length) {
      const img = document.createElement("img");
      img.src = URL.createObjectURL(this.files[0]);
      img.onload = function() {
        pcb = this.src
        setTimeout(() => {
          let pcbImg = document.getElementById('pcbImage')
          let imgWidth = pcbImg.clientWidth
          widthMultiplier = imgWidth/pcbWidth
        }, 100)
      }
    }
  }


</script>

<main>
  <div class="container">
    <div class="bom">
      <h1>Components</h1>
      <div class="components-filter">
        <button on:click={() => filterComponents('all')}>All</button>
        <button on:click={() => filterComponents('r')}>Resistor</button>
        <button on:click={() => filterComponents('c')}>Capacitor</button>
        <button on:click={() => filterComponents('ic')}>IC</button>
        <button on:click={() => filterComponents('others')}>Other</button>
      </div>
      <div class="components-list">
        {#each components as component}
          <div class="{component == selectedComponent ? 'selected' : ''} bom-component">
            <input on:change="{selectComponent}" type="radio" name="bom" id="{component.Designator}" value="{component.Designator}" />
            <label for="{component.Designator}">{component.Designator}: {component.Comment} <span>{component.Footprint}</span></label>
          </div>
        {/each}
      </div>
      
    </div>
    <div class="pcb">
      <div class="pcb-image">
        {#if !pcb}
          <input type="file" id="fileElem" on:change={handleFiles} multiple accept="image/*" style="display:none">
          <button on:click={selectFile} id="fileSelect">Select the pcb</button>
          <h3>No files selected yet.</h3>
        {:else}
          <img id="pcbImage" src={pcb} alt="The PCB" />
          <div class="component-overlay" style="left:calc({selectedComponent.mid_x * widthMultiplier}px - 4px);bottom:calc({selectedComponent.mid_y * widthMultiplier}px - 4px);transform:rotate({selectedComponent.Rotation}deg);"></div>
        {/if}
      </div>
      {#if Object.keys(selectedComponent).length}
        <div class="component-desc">
          <h2>Component: {selectedComponent.Designator} - {getType(selectedComponent.Designator)}</h2>
          <h2>Footprint: {selectedComponent.Footprint}</h2>
          <h2>Value: {selectedComponent.Comment}</h2>
        </div>
      {/if}
    </div>
  </div>

</main>

<style lang="scss">
  .container {
    display: grid;
    grid-template-columns: 1fr 3fr;
  }

  .components-list {
    max-height: 85vh;
    overflow: scroll;
  }
  
  .bom {
    padding: 10px;
    font-family: "Open sans", verdana;
    font-size: 18px;
    line-height: 1.5;
  }

  .bom-component {
    margin: 5px 0;
    padding: 3px;

    &.selected {
      background: linear-gradient(0deg, rgb(195 153 153) 0%, rgba(0,212,255,1) 17%, rgba(0,212,255,1) 100%);
    }

    display: grid;
    grid-template-columns: minmax(20px, 30px) auto;
    align-items: baseline;

    label {
      display: block;
      cursor: pointer;
    }

    span {
      font-size: 12px;
    }
  }

  .pcb {
    padding: 40px;
  }
  
  .pcb-image {
    position: relative;
    
    img {
      width: 100%;
    }
  }

  .components-filter {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    margin-bottom: 10px;

    button {
      display: inline;
      padding: 7px;
      font-size: 16px;
      cursor: pointer;
      background-color: teal;
      color: #FFF;
      border: none;

      &:not(:last-of-type) {
        border-right: 1px solid #FFF;
      }

      &:hover {
        background-color: lighten($color: teal, $amount: 8);
      }
    }
  }
  
  .component-overlay {
    position: absolute;
    display: block;
    width: 20px;
    height: 14px;
    background-color: red;
    border: 1px solid wheat;
    box-shadow: 1px 1px 4px black;
    animation: blink 1s infinite;
    left: -100vw;
  }

  @keyframes blink{
    0% {
      background: red;
    }
    20% {
      background: green;
    }
    40% {
      background: yellow;
    }
    60% {
      background: blue;
    }
    80% {
      background: orange;
    }
    100% {
      background: red;
    }
  }
</style>