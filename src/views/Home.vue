<template>

<div>

   <div class="section  bg-gray-200  border-b-4 border-black px-0 lg:px-1">

     <div class=" ">
        <Navbar 
        v-bind:web3Plug="web3Plug"
        
       />
     </div>


   </div>

  

   <div class="section  border-b-2 border-black" style="background:#1d1d1d;">
     <div class="text-white lg:flex lg:flex-row-reverse ">
       <div class=" w-full lg:w-1/2 p-8 ">
               

           <img src="@/assets/images/BananaSmasher.jpg" class=" py-8" style="margin:0 auto; width:100%;    " />
     


       </div>
       <div class=" w-full lg:w-1/2  text-center p-8 ">


     
               Mint dudes
        
          
       </div>


       


     </div>
       


   </div>


      
 

    
  <Footer/>

</div>
</template>


<script>



import Web3Plug from '../js/web3-plug.js'  
 
 
import Navbar from './components/Navbar.vue';
 
import Footer from './components/Footer.vue';
import TabsBar from './components/TabsBar.vue';
   
import FrontendHelper from '../js/frontend-helper.js';

const ERC721ABI = require('../contracts/ERC721ABI.json')


export default {
  name: 'Home',
  props: [],
  components: {Navbar, Footer },
  data() {
    return {
      web3Plug: new Web3Plug() , 
      signedInToWeb3: false,
      balances: {} ,
      tokenId: 0,
      donationAmount: 0 
      
    }
  },

  created(){

 
    this.web3Plug.getPlugEventEmitter().on('stateChanged', function(connectionState) {
        console.log('stateChanged',connectionState);
         
        this.activeAccountAddress = connectionState.activeAccountAddress
        this.activeNetworkId = connectionState.activeNetworkId 

        this.signedInToWeb3 =  (this.activeAccountAddress != null)
         
      }.bind(this));
   this.web3Plug.getPlugEventEmitter().on('error', function(errormessage) {
        console.error('error',errormessage);
         
        this.web3error = errormessage
       
      }.bind(this));

      this.web3Plug.reconnectWeb()
   
       

  },
  mounted: function () {
    this.getBalances()
    
    setInterval(  this.getBalances.bind(this), 5000  )
  }, 
  methods: {

          async getBalances(){

            const smasherAddress = '0xbf3122b2aa3102693e3194df7870e1a7ae146b50'
            
            const currencyAddress = '0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2' //WETH9  

            const currencyContract = this.web3Plug.getTokenContract( currencyAddress )

            let wethBalanceRaw = await currencyContract.methods.balanceOf( smasherAddress ).call()

            this.balances['WETH'] = this.web3Plug.rawAmountToFormatted(wethBalanceRaw, 18)

            console.log(' this.balances',  this.balances)

            this.$forceUpdate()

          },


          smash( ){
            

              let tokenId = parseInt( this.tokenId )    

                console.log('smash! ',tokenId)

              let userAddress = this.web3Plug.getActiveAccountAddress()

              const bananaContract = this.web3Plug.getCustomContract(  ERC721ABI , '0xb9ab19454ccb145f9643214616c5571b8a4ef4f2' )

              const currencyAddress = '0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2' //WETH9  

              const smasherAddress = '0xbf3122b2aa3102693e3194df7870e1a7ae146b50'

              bananaContract.methods.safeTransferFrom( userAddress, smasherAddress, tokenId , currencyAddress ).send({from: userAddress })
          },  
        

         donate( ){
            

              let donationAmount = this.web3Plug.formattedAmountToRaw( this.donationAmount, 18 )    

              

              let userAddress = this.web3Plug.getActiveAccountAddress()

            
              const currencyAddress = '0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2' //WETH9  

              const smasherAddress = '0xbf3122b2aa3102693e3194df7870e1a7ae146b50'

              const wethContract = this.web3Plug.getTokenContract(   currencyAddress  )


              wethContract.methods.transfer( smasherAddress, donationAmount ).send({from: userAddress })
          },  
 

  }
}
</script>
