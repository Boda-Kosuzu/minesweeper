<script>
  import Icon from 'fa-svelte'
  import { faFish } from '@fortawesome/free-solid-svg-icons'
  const fish = faFish
  import { onMount } from 'svelte'

  let trout = []
  let isEnded = false
  let isFinish = false

  onMount(() => {
    reset()
  })

  function reset() {
    trout = []
    isEnded = false
    isFinish = false
    
    for(let i = 0; i < 8; i++){
      const row = []
      for(let j = 0; j < 8; j++){
        row.push({
          isOpen: false,
          bomb: 0
        })
      }
      trout.push(row)
    }

    for(let i = 0; i < 10; i++) {
        let x = 0
        let y = 0
        while(true) {
        x = parseInt((Math.random() * 10) % 8)
        y = parseInt((Math.random() * 10) % 8)
        if (trout[y][x].bomb !== -1) {
          trout[y][x].bomb = -1
          break
        }
      }
      for(let a = -1; a <= 1; a++) {
        for(let b = -1; b <= 1; b++) {
          if(a === 0 && b ===0) continue
          if((y + a > -1 && y + a < 8) && (x + b > -1 && x + b < 8)) {
            if(trout[y + a][x + b].bomb === -1) continue
            trout[y + a][x + b].bomb += 1
          }
        }
      }
    }
  }

  function open(i, j) {
    if(isEnded || isFinish) return
    trout[j][i].isOpen = true
    if(trout[j][i].bomb === 0) {
      for(let a = -1; a <= 1; a++) {
        for(let b = -1; b <= 1; b++) {
          if(a === 0 && b ===0) continue
          if((j + a > -1 && j + a < 8) && (i + b > -1 && i + b < 8)) {
            if(trout[j + a][i + b].isOpen) continue
            trout[j + a][i + b].isOpen = true
            open(i + b, j + a)
          }
        }
      }
    } else if(trout[j][i].bomb === -1) {
      return true
    }
  }

  function isClear() {
    for(const row of trout) {
      for(const data of row) {
        if(!data.isOpen) {
          if(data.bomb !== -1) {
            isFinish = false
            return
          }
        }
      }
    }
    isFinish = true
  }
</script>

<style>
.minesweeper-area {
  font-size: 32px;
  border-spacing: 0;
  border-collapse: collapse;
  display: flex;
}

.tool-bar {
  display: flex;
  height: 70px;
  padding: 5px;
  box-sizing: border-box;
}

.end-message {
  font-size: 40px;
  color: rgb(228, 107, 107);
}

.clear-message {
  font-size: 40px;
  color: rgb(87, 153, 228);
}

.minesweeper-trout {
  width: 64px;
  height: 64px;
  background: #BDBDBD;
  box-sizing: border-box;
  display: flex;
  justify-content: center;
  align-items: center;
  user-select: none;
}

.close-trout {
  border-bottom: 3px solid #9E9E9E;
  border-right: 3px solid #9E9E9E;
  border-top: 3px solid #E0E0E0;
  border-left: 3px solid #E0E0E0;
  cursor: pointer;
}

.open-trout {
  border-top: 3px solid #9E9E9E;
  border-left: 3px solid #9E9E9E;
  border-bottom: 3px solid #E0E0E0;
  border-right: 3px solid #E0E0E0;
}

.title {
  margin: 0;
}
</style>

<div>
  <h1 class="title">マインスイーパー(マス)</h1>
  <div class="tool-bar">
    <button on:click={reset}>
      リセット
    </button>
    {#if isEnded}
    <label class="end-message">
      ぼかん
    </label>
    {/if}
    {#if isFinish}
    <label class="clear-message">
      クリア！！！！
    </label>
    {/if}
  </div>
  <div class="minesweeper-area">
    {#each trout as row, y}
      <div>
        {#each row as data, x}
          <div class="minesweeper-trout {data.isOpen? 'open-trout': 'close-trout'}"
            on:click={async () => {
              const result = await open(x, y)
              if(result) {
                isEnded = true
              } else {
                isClear()
              }
            }}
          >
            {#if (data.isOpen || isFinish || isEnded) && data.bomb !== 0}
              {#if data.bomb === -1}
                <Icon class="trout-icon" icon={fish} />
                {:else}
                  {data.bomb}
              {/if}
            {/if}
          </div>
        {/each}
      </div>
    {/each}
  </div>
</div>