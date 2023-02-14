<script lang=ts>
  import { createClient, gql, mutationStore, queryStore } from '@urql/svelte'
  import Record from './Record.svelte'

  const GQLClient = createClient({
    url: 'http://localhost:5000/graphql',
  })

  export let id

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
          {@const appRecord = record.node}
          <Record appRecord={appRecord} />
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
    background-color: darkgrey;
  }
</style>