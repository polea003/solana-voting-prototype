<template>
  <div class="w-screen h-screen bg-gray-700 text-white font-bold">
    <div class='flex flex-col space-y-4'>
      <div>Voting Project Prototype</div>
      <div v-if='!walletAddress' @click='connectWallet' class='bg-yellow-500 p-2 cursor-pointer bg-opacity-90 hover:bg-opacity-100'>Connect Wallet</div>
      <div v-if='!election && walletAddress' @click='createElectionAccount' class='bg-green-500 p-2 cursor-pointer bg-opacity-90 hover:bg-opacity-100'>Create Election</div>
      <div v-if='election && walletAddress' @click='vote(1)' class='bg-blue-500 p-2 cursor-pointer bg-opacity-90 hover:bg-opacity-100'>Vote for Candidate 1</div>
      <div v-if='election && walletAddress' @click='vote(2)' class='bg-red-500 p-2 cursor-pointer bg-opacity-90 hover:bg-opacity-100'>Vote for Candidate 2</div>
      <div v-for='(vote, index) in election' :key='vote'
        class='bg-green-400 p-2 bg-opacity-60'
      >
      {{`Vote ${index + 1}: Candidate ${vote.selection} by ${vote.userAddress}`}}
      </div>

    </div>
  </div>
</template>

<script>
import idl from './anchor_client.json';
import { Connection, PublicKey, clusterApiUrl } from '@solana/web3.js';
import { Program, Provider, web3 } from '@project-serum/anchor';

// SystemProgram is a reference to the Solana runtime!
const { SystemProgram, Keypair } = web3;

// Create a keypair for the account that will hold the GIF data.
let baseAccount = Keypair.generate();

// Get our program's id from the IDL file.
const programID = new PublicKey(idl.metadata.address);

// Set our network to devnet.
const network = clusterApiUrl('devnet');

// Controls how we want to acknowledge when a transaction is "done".
const opts = {
  preflightCommitment: "processed"
}

export default {
  name: 'App',
  async mounted () {
    const onLoad = async () => {
      await this.checkIfWalletIsConnected();
    };
    window.addEventListener('load', onLoad);
    return () => window.removeEventListener('load', onLoad);

  },
  data () {
    return {
      election: undefined,
      walletAddress: undefined
    }
  },
  methods: { 
    async checkIfWalletIsConnected () {
      try {
        const { solana } = window;
        if (solana) {
          if (solana.isPhantom) {
            console.log('Phantom wallet found!');
            // Logic to Connect Automatically if wallet trusts the site
            // Instead we will make a user connect their wallet everytime
            // const response = await solana.connect({ onlyIfTrusted: true });
            // console.log(
            //   'Automaticlly Connected with Public Key:',
            //   response.publicKey.toString()
            // );
            // this.walletAddress = response.publicKey.toString();
          }
        } else {
          alert('Solana object not found! Get a Phantom Wallet 👻');
        }
      } catch (error) {
        console.error(error);
      }
    },
    async connectWallet () {
      const { solana } = window;
      if (solana) {
        const response = await solana.connect();
        console.log('Sucessfully Connected with Public Key:', response.publicKey.toString());
        this.walletAddress = response.publicKey.toString();
      }
    },
    getProvider () {
      const connection = new Connection(network, opts.preflightCommitment);
      const provider = new Provider(
        connection, window.solana, opts.preflightCommitment,
      );
      return provider;
    },
    async getElection () {
      try {
        const provider = this.getProvider();
        const program = new Program(idl, programID, provider);
        const account = await program.account.baseAccount.fetch(baseAccount.publicKey);
        
        console.log("Got the account", account)
        this.election = [...account.votes]
      } catch (error) {
        console.log("Error in getElection: ", error)
        this.election = null;
      }
    },
    async createElectionAccount() {
      try {
        const provider = this.getProvider();
        const program = new Program(idl, programID, provider);
        console.log("ping")
        await program.rpc.initialize({
          accounts: {
            baseAccount: baseAccount.publicKey,
            user: provider.wallet.publicKey,
            systemProgram: SystemProgram.programId,
          },
          signers: [baseAccount]
        });
        console.log("Created a new BaseAccount w/ address:", baseAccount.publicKey.toString())
        await this.getElection();
      } catch(error) {
        console.log("Error creating BaseAccount account:", error)
      }
    },
    async vote (selection) {
        try {
          const provider = this.getProvider();
          const program = new Program(idl, programID, provider);
          await program.rpc.addVote(selection, {
            accounts: {
              baseAccount: baseAccount.publicKey,
              user: provider.wallet.publicKey,
            },
          });
          console.log("Vote successfully sent to program:", 'selection =', selection )
          await this.getElection();
        } catch (error) {
          console.log("Error sending Vote:", error)
        }   
    }
  }

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
