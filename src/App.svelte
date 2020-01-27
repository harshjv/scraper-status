<script>
import { onMount } from 'svelte'

const formatter = new Intl.NumberFormat()
const results = {
	mainnet: {},
	rinkeby: {},
	kovan: {}
}

const random = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min

async function run (network) {
	const response = await fetch(`https://${network}-ethereum.harshjv.com/status`)
  const json = await response.json()
	const { status } = json.data

	if (status.difference <= 14) {
		status.level = 'success'
	} else if (status.difference > 14 && status.difference <= 25) {
		status.level = 'warning'
	} else {
		status.level = 'danger'
	}

	status.difference = formatter.format(status.difference)
	status.latestScrapedBlockNumber = formatter.format(status.latestScrapedBlockNumber)
	status.latestBlockNumber = formatter.format(status.latestBlockNumber)

	results[network] = status

	setTimeout(run, random(3500, 5000), network)
}

function start () {
	Object.keys(results).forEach(run)
}

onMount(start)
</script>

<main>
	<div class="container">
		<div class="row">
		{#each Object.entries(results) as [network, status]}
		<div class="col-lg-4 col-md-4 col-12">
			<div class="card">
			  <div class="card-body">
			    <h5 class="card-title">
						<span>{network}</span>
						<small class="badge badge-{status.level}" title="Block Gap">{status.difference ? status.difference : '...'}</small>
					</h5>
			    <h6 class="display-4">{status.latestScrapedBlockNumber ? status.latestScrapedBlockNumber : '...'}</h6>
			    <p class="card-text small text-muted">Latest block is {status.latestBlockNumber ? status.latestBlockNumber : '...'}</p>
			  </div>
			</div>
		</div>
		{/each}
		</div>
	</div>
</main>

<style>
main {
	height: 100%;
	display: flex;
	justify-content: center;
  align-items: center;
}

.card-title {
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.card-title small {
	font-size: 0.85rem;
	font-weight: 400;
}
</style>
