<script lang="ts">
  import { createClient, gql, setContextClient, queryStore, getContextClient } from '@urql/svelte'

  const client = createClient({
    url: 'http://localhost:5000/graphql',
  })

  setContextClient(client)

  const noteStore = queryStore({
    client: getContextClient(),
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
                  }
                }
              }
            }
          }
        }
      }
    `,
  })

  //let edges: any

</script>

<p>D14-N3</p>

<ul>
  {#if $noteStore.fetching} <li>Loading...</li>
  {:else if $noteStore.error} <li>ERROR: {$noteStore.error.message}</li>

  {:else}

    {@const edges = $noteStore.data?.allAppNotes?.edges}
    
		{#each edges as edge (edge.node.id)}
			<p>{edge.node.note}</p>
		{/each}

  {/if}
</ul>