<script>
  import { Modal, ModalContent, Input, Button } from "@budibase/bbui"

  export let addArchers

  let modal

  // ARCHERS

  const resetArchers = () => {
    return [{name: "", score: 0}]
  }

  let archers = resetArchers()

  const confirm = () => {
    addArchers(archers)
    archers = resetArchers()
    modal.hide()
  }

  // MODAL ACTIONS

  export let onDismiss = () => {
    archers = resetArchers()
  }

  export let onShow = () => {}

  export function show() {
    modal.show()
  }

  export function hide() {
    modal.hide()
  }

</script>

<Modal bind:this={modal} on:show={onShow} on:hide={onDismiss} onConfirm={confirm}>
    <ModalContent
            title={"Add Archers"}
            size="M"
            confirmText="Add"
            onConfirm={() => addArchers(archers)}
    >
        {#each archers as archer}
            <Input bind:value={archer.name} label="Name"></Input>
        {/each}
        <div>
            <Button
                newStyles
                secondary
                on:click={() => {
              archers = [...archers, {name: "", score: 0}]
            }}
            >
                New player
            </Button>
        </div>

    </ModalContent>
</Modal>

<style>

</style>
