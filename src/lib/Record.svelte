<script lang=ts>
  export let appRecord: any
  console.log('app ', appRecord)
</script>

<li>
  <h3>{appRecord.name}</h3>
  <p>Recorded on: {appRecord.date}</p>
  <p>Path: {appRecord.path}</p>
</li>


