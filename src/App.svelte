<script>
  let pcbWidth
  let selectedComponent = {}
  let selectedFilter = 'all'
  let components = []
  let availableComponents = []
  let pnp = []
  let widthMultiplier
  let imgWidth
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

  let pnpCsv

  const selectComponent = (e) => {
    const selected = e.currentTarget.value
    selectedComponent = pnp.filter(c => c.designator == selected)[0]
  }

  const filterComponents = (f) => {
    switch (f) {
      case 'r':
        components = pnp.filter(c => c.designator.charAt(0).toLowerCase() == 'r')
        selectedFilter = 'resistor'
        break;
      case 'c':
        components = pnp.filter(c => c.designator.charAt(0).toLowerCase() == 'c')
        selectedFilter = 'capacitor'
        break;
      case 'ic':
        components = pnp.filter(c => ['q', 'u'].includes(c.designator.charAt(0).toLowerCase()))
        selectedFilter = 'ic'
        break;
      case 'others':
        components = pnp.filter(c => !['c', 'r', 'q', 'u'].includes(c.designator.charAt(0).toLowerCase()))
        selectedFilter = 'others'
        break;
      default:
        components = pnp
        selectedFilter = 'all'
        break;
    }
  }

  $: layer = selectedComponent.layer ? selectedComponent.layer.toLowerCase() : 't'
  $: layerPos = (selectedComponent.layer && selectedComponent.layer.toLowerCase() === 't') ? 'left' : 'right'

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
          imgWidth = pcbImg.clientWidth
          widthMultiplier = imgWidth/pcbWidth
        }, 100)
      }
    }
  }

  const csvJSON = (csv) => {
    let lines=csv.split("\n")
    let result = []
    let headers=lines[0].split("\t");
    for(let i=1; i<lines.length; i++) {
        let obj = {}
        if (lines[i]) {
        let currentline=lines[i].split("\t")
        for(let j=0; j<headers.length; j++) {
          let header = headers[j].replaceAll(' ', '_').toLowerCase()
          obj[header] = currentline[j].replaceAll('"', '').replaceAll('mm', '')
        }
        result.push(obj)
    }
    }
    return result
  }

  const uploadPnp = () => {
    let csvVal = csvJSON(pnpCsv)
    if (Array.isArray(csvVal)) {
      components = pnp = csvVal
      components.map(c => {
        if (c.designator.charAt(0).toLowerCase() == 'r') {
          !availableComponents.includes('resistor') ? availableComponents.push('resistor') : ''
        }
        if (c.designator.charAt(0).toLowerCase() == 'c') {
          !availableComponents.includes('capacitor') ? availableComponents.push('capacitor') : ''
        }
        if (['q', 'u'].includes(c.designator.charAt(0).toLowerCase())) {
          !availableComponents.includes('ic') ? availableComponents.push('ic') : ''
        }
      })
    }
    console.log(csvVal)
  }


</script>

<main>
  <div class="container">
    <div class="bom">
      <h1>Components</h1>
      {#if pnp.length}
        <div class="components-filter">
          <button on:click={() => filterComponents('all')} class:selected={selectedFilter == 'all'}>All</button>
          {#if availableComponents.includes('resistor')}
            <button on:click={() => filterComponents('r')} class:selected={selectedFilter == 'resistor'}>Resistor</button>
          {/if}
          {#if availableComponents.includes('capacitor')}
            <button on:click={() => filterComponents('c')} class:selected={selectedFilter == 'capacitor'}>Capacitor</button>
          {/if}
          {#if availableComponents.includes('ic')}
            <button on:click={() => filterComponents('ic')} class:selected={selectedFilter == 'ic'}>IC</button>
          {/if}
          <button on:click={() => filterComponents('others')} class:selected={selectedFilter == 'others'}>Other</button>
        </div>
        <div class="components-list">
          {#each components as component}
            <div class="{component == selectedComponent ? 'selected' : ''} bom-component">
              <input on:change="{selectComponent}" type="radio" name="bom" id="{component.designator}" value="{component.designator}" />
              <label for="{component.designator}">{component.designator}: {component.comment} <span>{component.footprint}</span></label>
            </div>
          {/each}
        </div>
      {:else}
        <div class="upload-pnp">
          <textarea bind:value={pnpCsv} name="pnpFile" id="pnpFile" cols="30" rows="10" placeholder="Paste CSV content you've exported from PCB design software here and click on upload button"></textarea>
          <button class="btn btn-wide" on:click|preventDefault={uploadPnp}>Upload</button>
        </div>
      {/if}
      
    </div>
    <div class="pcb">
      <div class="pcb-image">
        {#if !pcb}
          <input type="file" id="fileElem" on:change={handleFiles} multiple accept="image/*" style="display:none">
          <input type="text" bind:value={pcbWidth}>
          <button on:click={selectFile} id="fileSelect">Select the pcb</button>
          <h3>No files selected yet.</h3>
        {:else}
          <div class="pcb-container" style="width: {imgWidth}px;">
            <img id="pcbImage" src={pcb} alt="The PCB" />
            <div class="component-overlay" style="{layerPos}:calc({selectedComponent.mid_x * widthMultiplier}px - 8px);bottom:calc({selectedComponent.mid_y * widthMultiplier}px - 4px);transform:rotate({selectedComponent.rotation}deg);"></div>
          </div>
        {/if}
      </div>
      {#if Object.keys(selectedComponent).length}
        <div class="component-desc">
          <h2>Component: {selectedComponent.designator} - {getType(selectedComponent.designator)}</h2>
          <h2>Footprint: {selectedComponent.footprint}</h2>
          <h2>Value: {selectedComponent.comment}</h2>
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

  .upload-pnp {
    display: flex;
    flex-direction: column;

    button {
      padding-top: 7px;
      padding-bottom: 7px;
      font-size: 20px;
      cursor: pointer;
      background-color: teal;

      &:hover {
        background-color: lighten($color: teal, $amount: 8);
      }
    }
  }

  .components-list {
    max-height: 85vh;
    overflow: scroll;
  }
  
  .bom {
    padding: 20px 10px;
    font-family: "Open sans", verdana;
    font-size: 18px;
    line-height: 1.5;

    h1 {
      margin-top: 0;
    }
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
    padding: 20px 40px;
  }
  
  .pcb-container {
    position: relative;
    overflow: hidden;
    
    img {
      width: auto;
      max-width: 100%;
      max-height: 90vh;
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

      &:hover,
      &.selected {
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