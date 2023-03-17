<script lang="ts">
	import SubmitForm from "$lib/SubmitForm.svelte";
	import "svelte-schema-form/css/layout.scss";
	import "svelte-schema-form/css/basic-skin.scss";
	import type { ValidationErrors } from "$lib/types/CommonComponentParameters";
  import { opacAddresses } from "../lib/opacAddresses";
	import set from "lodash-es/set";
	import get from "lodash-es/get";

	let schema: any = JSON.parse(`{
		"type": "object",
		"properties": {
			"walletAddress": {
				"type": "string",
				"suggestions": [
					"0x1ABC7154748D1CE5144478CDEB574AE244B939B5",
          "0xbf4f36efa3ac655a1d86f6c32b648a90271443f4",
					"mrblobby.eth"
				],
				"editor": "suggester"
			},
			"token": {
				"type": "string",
				"enum": [
					"ETH",
					"USDC",
					"ANT"
				]
			},
			"amount": {
				"type": "number"
			}
		},
		"required": [ "walletAddress", "token", "amount" ]
	}`);
	let storedSchema = undefined;
	if (typeof window !== 'undefined') {
		storedSchema = window.localStorage.getItem('schema');
		if (storedSchema) schema = JSON.parse(storedSchema);
	}
	let jsonInvalid = false;

	let value = { };
	let valueJson = '';
	let collapsible = false;

	const submit = (e: CustomEvent) => {
		valueJson = JSON.stringify(e.detail.value, undefined, 2).trim();
	};

	const onValue = ({detail}: CustomEvent) => {
		console.log('onval: ' + JSON.stringify(detail));
		const path = (detail.path as string[]).join('.');
		const errors = detail.errors as ValidationErrors;
		let value = detail.value.walletAddress;
		const prevWallets = schema.properties.walletAddress.suggestions;
		if (!prevWallets.includes(value) && !errors["prevWallets"]) {
			errors["walletAddress"] = "Caution: This wallet address is newly created and has not been used before. Scammers frequently employ fresh wallet addresses for their schemes. Please exercise vigilance and verify the address's authenticity before proceeding with any transactions.";
		}

		value = detail.value.token;
		if (value === "ANT" && !errors["token"]) {
			errors["token"] = "Caution: Governance tokens sent may impact DAO management and community control";
		}

		value = detail.value.amount;
		if (detail.value.token === 'ANT' && !errors["amount"]) {
			if ((value as number) > 20) {
				errors["amount"] = "Attention: You are sending over 20% of your governance tokens";
			}
		} else {
			if ((value as number) > 33) {
				errors["amount"] = "Heads up: You are Transferring over 33% of your tokens"
			}
		}

    value = detail.value.walletAddress;
    console.log({ isItIncluded: opacAddresses.includes(value) });
    if (opacAddresses.includes(value) && !errors["walletAddress"]) {
      errors["walletAddress"] = "Caution: This wallet address is an OFAC address. Please verify the authenticity of the address before proceeding with any transactions.";
    };

	}

	const componentContext = { currencySymbol: '£' };
</script>

<div class="container">
	<div class="form">
		<SubmitForm {schema} {value} on:submit={submit} on:value={onValue} uploadBaseUrl="https://restspace.local:3131/files" {collapsible} {componentContext} />
	</div>
	<div class="output">
		<pre>
			{valueJson}
		</pre>
	</div>
</div>

<style>
	* {
		box-sizing: border-box;
	}

	.container {
		display: flex;
		position: relative;
	}
	.form, .output {
		width: 49%;
		border: solid 1px black;
		height: 99vh;
		position: relative;
	}
	.schema {
		border: none;
		display: flex;
		flex-direction: column;
	}
	.form, .output {
		margin-left: 1%;
		padding: 1em;
	}
	#schema {
		width: 100%;
		height: 100%;
		gap: 1em;
	}
	.schema.jsonInvalid #schema {
		color: darkred;
	}

	#collapsible {
		margin-bottom: 6px;
	}

	.control {
		margin-bottom: 6px;
	}
</style>
