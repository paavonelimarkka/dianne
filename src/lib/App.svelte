<script lang="ts">
  import { onMount } from 'svelte'
  import { createClient, gql, setContextClient, mutationStore, queryStore, getContextClient } from '@urql/svelte'
	import { KnownArgumentNamesRule } from 'graphql';

  const GQLclient = createClient({
    url: 'http://localhost:5000/graphql',
  })

  //setContextClient(client)
  

  const noteStore = queryStore({
    client: GQLclient,
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

  let noteName = ''
  let formSubmitted: boolean = false
  
  
  const createNote = (note: string, kala: any) => {
    mutationStore({
      client: GQLclient,
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
    formSubmitted = false
  }

$: if (formSubmitted) {
  createNote(noteName)
}
  

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

        <li>
          <h2>{edge.node.note} by {edge.node.appUserByUserId?.username}</h2>

            {#if edge.node.appRecordsByNotesId.edges}
              {@const records = edge.node.appRecordsByNotesId.edges}
              <ul>

                {#each records as record (record.node.id) }
                  <li>
                    <h3>{record.node.name}</h3>
                    <p>Recorded on: {record.node.date}</p>
                    <p>Path: {record.node.path}</p>
                  </li>
                {/each}
              </ul>
            {/if}
          
        </li>
      {/each}
  
    {/if}
  </ul>

  <form>
    <input type="text" bind:value={noteName} placeholder="Enter your name" />
    <button type="button" on:click={() => formSubmitted=true }>Submit</button>
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
