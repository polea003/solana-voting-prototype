<template>
  <div class="w-screen h-screen bg-gray-700 text-white">
    <div class='flex flex-col space-y-4'>
      <div>Voting Project Prototype</div>
      <div class='bg-green-500 p-2'>Create Election</div>
      <div class='bg-blue-500 p-2'>Vote for Candidate 1</div>
      <div class='bg-red-500 p-2'>Vote for Candidate 2</div>

    </div>
  </div>
</template>

<script>

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
          }
        } else {
          alert('Solana object not found! Get a Phantom Wallet 👻');
        }
      } catch (error) {
        console.error(error);
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
