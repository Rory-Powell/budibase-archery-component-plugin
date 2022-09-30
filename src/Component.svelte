<script>
  import { getContext } from "svelte"
  import { Button, Table, Heading, Body } from "@budibase/bbui"
  import Target from "./Target.svelte"
  import Arrow from "./Arrow.svelte"
  import AddArchersModal from "./AddArchersModal.svelte"

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  export let targetSize
  export let strokeWidth
  export let targetType
  export let arrowRadius
  export let onSaveResults

  // ARROWS

  let arrows = []

  const addArrow = (arrow) => {
    arrows = [...arrows, arrow]
  }

  const removeLastArrow = () => {
    arrows.pop()
    arrows = [...arrows]
  }

  // ARCHERS

  let addArchersModal
  $: archers = []
  $: activeArcher = 0

  const showArchersModal = () => {
    addArchersModal.show()
  }

  const archerSchema = {
    name: {},
    score: {}
  }

  const addArchers = (newArchers) => {
    archers = [...archers, ...newArchers]
  }

  // RESULTS

  const saveResults = () => {
    const results = archers
    console.log({ results })
    onSaveResults({
      results
    })
  }

  // GAME

  $: started = false
  $: end = 0

  const progressGame = async () => {
    // start the game
    if (!started) {
      started = true
      activeArcher = 0
      end = end + 1
      return
    }

    // add the new score
    let score = 0
    for (let arrow of arrows) {
      score = arrow.score + score
    }
    archers[activeArcher].score = archers[activeArcher].score + score
    archers = [...archers]

    // reset for next archer
    activeArcher = activeArcher + 1
    if (activeArcher === archers.length) {
      end = end + 1
      activeArcher = 0
    }
    arrows = []
  }

</script>


<div use:styleable={$component.styles}>
  <AddArchersModal bind:this={addArchersModal} addArchers={addArchers}/>
  <Target
      canvasSizePx={targetSize || 500}
      strokeWidth={strokeWidth || 1}
      targetType={targetType || "olympic"}
      arrowRadius={arrowRadius | 6}
      arrows={arrows}
      addArrow={addArrow}
  />
  <div>
    {#if arrows.length === 0 && started}
      <p
        class="centered-text"
        style={{ fontWeight: 'bold', paddingTop: '10px' }}
      >
        No arrows yet
      </p>
    {/if}
      <div class="arrow-container">
        {#each arrows as arrow, index}
            <Arrow
              arrow={arrow}
              key={index}
              targetType={targetType}
            />
        {/each}
      </div>
    <div class="buttons">
      {#if started}
        <div class="button">
          <Button
              newStyles
              on:click={() =>
              addArrow({
                point: { x: -1, y: -1 },
                score: 0
              })
            }
          >
            Miss
          </Button>
        </div>

        <div class="button">
          <Button
            newStyles
            on:click={() => removeLastArrow()}
            warning
          >
            Undo
          </Button>
        </div>
      {/if}

      {#if archers.length}
        <div class="button">
          <Button
            newStyles
            cta
            disabled={archers.length === 0}
            on:click={() => progressGame()}
          >
            {#if started}
              Next
            {:else}
              Start
            {/if}

          </Button>
        </div>
      {/if}
    </div>
    <div class="archers-heading">
      <Heading
            weight="default"
      >
        Archers
      </Heading>
      {#if started && archers.length}
        <Body>
          Archer: {archers[activeArcher].name}
        </Body>

      {/if}
      {#if !!end}
        <Body>
        End: #{end}
        </Body>

      {/if}
      {#if !started}
      <Button
        icon="Add"
        newStyles
        secondary
        on:click={() => showArchersModal()}
      >Add archer</Button>
      {/if}

    </div>
    <div>
      <Table
          data={archers}
          schema={archerSchema}
          placeholderText="No archers"
          allowEditColumns={false}
          allowEditRows={false}
      >
      </Table>
    </div>
  </div>
  {#if started}
  <div class="finished">
    <Button on:click={saveResults}>
      Save Results
    </Button>
  </div>
  {/if}
</div>

<style>
  .centered-text {
    text-align: center !important;
  }
  .arrow-container {
    margin-top: 20px;
    flex-wrap: wrap;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .buttons {
    margin-top: 20px;
    margin-bottom: 30px;
    display: flex;
    justify-content: center;
  }
  .button {
    margin-right: 10px;
  }
  .archers-heading {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
    margin-bottom: 10px;
  }
  .finished {
    margin-top: 20px;
  }
</style>