<script lang=ts>
  import { createClient, gql, setContextClient, mutationStore, queryStore } from '@urql/svelte'

  const GQLClient = createClient({
    url: 'http://localhost:5000/graphql',
  })

  export let id

  /*
  const noteStore = queryStore({
    client: GQLClient,
    query: gql`
      query AllAppNotesQuery {
        appNoteById {
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
  */

  const appNote = queryStore({
    client: GQLClient,
    query: gql`
      query AppNote ($id: Int!) {
        appNoteById(id: $id) {
          id
          nodeId
          note
          userId
          appRecordsByNotesId {
            edges {
              node {
                date
                id
                name
                nodeId
                notesId
                path
              }
            }
          }
          appUserByUserId {
            username
          }
        }
      }
    `,
    variables: { id }
  })

  $: deleteNote = ( id: number ) => {
    mutationStore({
      client: GQLClient,
      query: gql`
        mutation MyMutation2 ($id: Int!) {
          deleteAppNoteById(input: {id: $id}) {
            deletedAppNoteId
          }
        }
      `,
      variables: { id }
    })
  }

  console.log('tänne noten puolelle tuleva id ois: ', id)

</script>


{#if $appNote.fetching}
  <li>Loading...</li> 
{:else if $appNote.error}
  <li>ERROR: {$appNote.error.message}</li>
{:else}

{@const note = $appNote.data?.appNoteById}

  <article class="note">

    <h2>{note.note} by {note.appUserByUserId.username}</h2>

    {#if note.appRecordsByNotesId.edges}
    {@const records = note.appRecordsByNotesId.edges}
      
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

    <form>
      <button type="button" on:click={() => {
        deleteNote(note.id)
      }}>Delete note</button>
    </form>

  </article>

{/if}

<style>
  .note {
    background-color: cadetblue;
  }
</style>