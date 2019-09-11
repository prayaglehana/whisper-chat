<template>
	<div>
		<h1>Whisper Example Chat Application</h1>
		<!-- <button @click="get_key_id_pair_from_pvt_key('0x80153F513EA70A9E0A32AB59D0D61281DA769D985D148B3140FF7AB1356D899F')">get keyid from pvtkey</button> -->
		<div v-if="!configured">
			enter your key-id pair : <input v-model="asymKeyId" @keyup.enter="extractPUBKEY" />
		
			<asymmetric-key-config :pub-key="asymPubKey" :key-id="asymKeyId"></asymmetric-key-config>

			username: <input v-model="name" /><br>
			<button @click="configWithKey" >Start</button>
		</div>
		<div v-else>
			<div>
				My publick key: {{asymPubKey}}
				Recipient's public key: <input  v-model="recipientPubKey" />
			</div>
	
			<p v-for="(m,i) of msgs" v-bind:key="i" >
				<b>{{m.name}}</b>: {{m.text}}
			</p>
			Please type a message: <input v-model="text" @keyup.enter="sendMessage" />
			<button @click="sendMessage">Send</button>

		</div>
	</div>
</template>

<script>
import Web3 from 'web3';
import AsymmetricKeyConfig from './AsymmetricKeyConfig.vue';
import {decodeFromHex, encodeToHex} from './hexutils';

const defaultRecipientPubKey = "";
const defaultTopic = "0x5a4ea131";

const sec=""

export default {
	data() {
		this.web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
		this.shh = this.web3.shh;

		let data = {
			msgs: [],
			text: "",
			symKeyId: null,
			name: "",
			asymKeyId: null,
			sympw: "",
			configured: false,
			topic: defaultTopic,
			recipientPubKey: defaultRecipientPubKey,
			asymPubKey: ""
		};

		// this.shh.newKeyPair().then(id => {
		// 	console.log('dfsdddsdf');
			
		// 	//this.asymKeyId=
		// 	data.asymKeyId = id;
		// 	return this.shh.getPublicKey(id).then(pubKey => this.asymPubKey = pubKey).catch(console.log);

		// }).catch(console.log);


		return data;
	},

	components: {AsymmetricKeyConfig},

	methods: {
		
	
		sendMessage() {
			let msg = {
				text: this.text,
				name: this.name
			};

			this.msgs.push(msg);

			let postData = {
				ttl: 7,
				topic: '0x07678231',
				powTarget: 2.01,
				powTime: 100,
				payload: encodeToHex(JSON.stringify(msg)),
			};

			postData.pubKey = this.recipientPubKey;
			postData.sig = this.asymKeyId;
			
			this.shh.post(postData);

			this.text = "";
		},
		extractPUBKEY(){
				console.log(this.shh);
					
					this.shh.getPublicKey(this.asymKeyId).then(pubKey =>{
							this.asymPubKey=pubKey	;
							console.log('pub key extracted',this.asymPubKey);

					} );
		},
		get_key_id_pair_from_pvt_key(pvtkey){
			this.shh.addPrivateKey(pvtkey)
					.then(id=>{
						console.log(id);
					}).catch(console.log);
		},
		
		configWithKey() {
			// TODO use a form
			if (!this.name || this.name.length == 0) {
				alert("Please pick a username");
				return;
			}

			let filter = {
				topics: ['0x07678231']
			};

			if(!this.asymKeyId) {
					alert("No valid asymmetric key",this.asymKeyId);
				return;
			}

		    filter.privateKeyID = this.asymKeyId;
			
			this.msgFilter = this.shh.newMessageFilter(filter).then(filterId => {
				setInterval(() => {
					console.log('setInter');
					this.shh.getFilterMessages(filterId).then(messages => {
						console.log('filter donr');
						for (let msg of messages) {
							console.log('inside');
							let message = decodeFromHex(msg.payload);
							this.msgs.push({
								name: message.name,
								text: message.text
							});
						}
					});
				}, 1000);
			});

			this.configured = true;
		}
	}
};
</script>
