<script lang="ts">
  import { createClient, gql, setContextClient, mutationStore, queryStore } from '@urql/svelte'
  import Note from './Note.svelte'

  const GQLClient = createClient({
    url: 'http://localhost:5000/graphql',
  })

  const noteStore = queryStore({
    client: GQLClient,
    query: gql`
      query AllAppNotesQuery {
        allAppNotes {
          edges {
            node {
              id
              note
              appRecordsByNotesId {
                edges {
                  node {
                    id
                    name
                    path
                    nodeId
                    date
                  }
                }
              }
              appUserByUserId {
                username
                nodeId
              }
            }
          }
        }
      }
    `,
  })

  $: createNote = (note: string) => {
    mutationStore({
      client: GQLClient,
      query: gql`
        mutation MyMutation($note: String!) {
          createAppNote(input: {appNote: {userId: 1, note: $note}}) {
            appNote {
              note
            }
          }
        }
      `,
      variables: { note }
    })
  }
  
  let noteName = ''

</script>


<header>
  <h1>Di-4N3</h1>
</header>

<main>

  <ul>

    {#if $noteStore.fetching} <li>Loading...</li>
    {:else if $noteStore.error} <li>ERROR: {$noteStore.error.message}</li>
    {:else}

      {@const edges = $noteStore.data?.allAppNotes?.edges}
      {#each edges as edge (edge.node.id)}
      {@const id = edge.node.id}

        <li>
          <Note id={edge.node.id} />
        </li>

      {/each}
    {/if}

  </ul>

  <form>
    <input type="text" bind:value={noteName} placeholder="Enter your name" />
    <button type="button" on:click={() => createNote(noteName)}>Submit</button>
  </form>

</main>


<style>

  main, header {
    text-align: center;
  }

  ul {
    list-style: none;
  }

</style>
