<script lang=ts>
  import App from '../lib/App.svelte'
  import { createClient, gql, setContextClient, queryStore, getContextClient } from '@urql/svelte'

  const client = createClient({
    url: 'http://localhost:5000/graphql',
  })

  setContextClient(client)

  const data = queryStore({
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
                  }
                }
              }
            }
          }
        }
      }
    `,
  })

  console.log(data)

</script>

<App />
