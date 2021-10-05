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
               

           <img v-bind:src="encodedImageSVG" class=" py-8" style="margin:0 auto; width:40%;    " />
     


       </div>
       <div class=" w-full lg:w-1/2  text-center p-8 " v-if="canMint">


            <div>
               <div class="text-white text-xl">
                    Mint your on-chain 0xBTC stats NFT
              </div>
                <div class="text-white text-xs">
                   Updates in realtime to show current supply, difficulty, and reward. 
              </div>

            </div>

              <div class="my-16 text-center">
                <div class="text-white text-xs">
                   Cost per NFT: 0.02 ETH
                 </div>


                <div  class="flex flex-row">
                  <div class="flex-grow"></div>
                  <input type="number" min=1 max=10 v-model="mintAmount" class="text-black border-black border-2 p-2 mx-4 " />
                  <div class="select-none bg-orange-500 hover:bg-orange-300 p-2 px-8 rounded border-gray border-2 cursor-pointer" @click="mint"> Mint </div>
                  <div class="flex-grow"></div>
              </div>  

            </div>

            <div>

               <div class="text-white text-xs ">
                    {{ errorMessage }} 
              </div>


               <div class="text-white text-xs my-2">
                    {{ totalSupply }} minted of 420 max.
              </div>
                 

            </div>

        
          
       </div>

        <div class=" w-full lg:w-1/2  text-center p-8 " v-if="!canMint">

             <div>
               <div class="text-white text-xl">
                    This collection has been completely minted!
              </div>
               

            </div>



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
const StatsNFTABI = require('../contracts/StatsNFTABI.json')


export default {
  name: 'Home',
  props: [],
  components: {Navbar, Footer },
  data() {
    return {
      web3Plug: new Web3Plug() , 
      signedInToWeb3: false,
      balances: {} ,
      totalSupply: 0,
      mintAmount: 1,
      errorMessage: null,

      encodedMetadata: "data:application/json;base64,eyJuYW1lIjogIjB4QlRDIFN0YXRzICMwIiwgImRlc2NyaXB0aW9uIjogIk1pbmVhYmxlIHRva2VuIHN0YXRpc3RpY3MuIiwgImltYWdlIjogImRhdGE6aW1hZ2Uvc3ZnK3htbDtiYXNlNjQsUEhOMlp5QjRiV3h1Y3owaWFIUjBjRG92TDNkM2R5NTNNeTV2Y21jdk1qQXdNQzl6ZG1jaUlIQnlaWE5sY25abFFYTndaV04wVW1GMGFXODlJbmhOYVc1WlRXbHVJRzFsWlhRaUlIWnBaWGRDYjNnOUlqQWdNQ0F6TlRBZ016VXdJajQ4YzNSNWJHVStMbUpoYzJVZ2V5Qm1hV3hzT2lCM2FHbDBaVHNnWm05dWRDMW1ZVzFwYkhrNklITmxjbWxtT3lCbWIyNTBMWE5wZW1VNklERTJjSGc3SUgwOEwzTjBlV3hsUGp4eVpXTjBJSGRwWkhSb1BTSXhNREFsSWlCb1pXbG5hSFE5SWpFd01DVWlJR1pwYkd3OUltSnNZV05ySWlBdlBqeDBaWGgwSUhnOUlqRXdJaUI1UFNJeU1DSWdZMnhoYzNNOUltSmhjMlVpUGkwdExTQXdlRUpVUXlCVGRHRjBjeUF0TFMwOEwzUmxlSFErUEhSbGVIUWdlRDBpTVRBaUlIazlJalF3SWlCamJHRnpjejBpWW1GelpTSStUV2x1WldRZ1UzVndjR3g1T2lBMU1EQXdQQzkwWlhoMFBqeDBaWGgwSUhnOUlqRXdJaUI1UFNJMk1DSWdZMnhoYzNNOUltSmhjMlVpUGsxcGJtbHVaeUJFYVdabWFXTjFiSFI1T2lBeFBDOTBaWGgwUGp4MFpYaDBJSGc5SWpFd0lpQjVQU0k0TUNJZ1kyeGhjM005SW1KaGMyVWlQazFwYm1sdVp5QlNaWGRoY21RNklEVXdQQzkwWlhoMFBqd3ZjM1puUGc9PSJ9",
      encodedImageSVG: null
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
   
     this.parseMetadata() 

  },
  mounted: function () {
    this.getBalances()


    this.getTotalSupply()

    
    setInterval(  this.getBalances.bind(this), 5000  )
  }, 
  methods: {
          canMint(){
            return this.totalSupply >= 420
          },
    

          async getBalances(){
/*
            const smasherAddress = '0xbf3122b2aa3102693e3194df7870e1a7ae146b50'
            
            const currencyAddress = '0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2' //WETH9  

            const currencyContract = this.web3Plug.getTokenContract( currencyAddress )

            let wethBalanceRaw = await currencyContract.methods.balanceOf( smasherAddress ).call()

            this.balances['WETH'] = this.web3Plug.rawAmountToFormatted(wethBalanceRaw, 18)

            console.log(' this.balances',  this.balances)

            this.$forceUpdate()
*/  
          },

          async getTotalSupply(){

            let contractData = await this.web3Plug.getContractDataForActiveNetwork()

            const nftContract = this.web3Plug.getCustomContract( StatsNFTABI, contractData.statsnft.address  )

            this.totalSupply = await nftContract.methods.totalSupply().call()
  

            this.$forceUpdate()
 
          },


          parseMetadata( metadata ){
            let tokenURI = this.encodedMetadata 

            const split = tokenURI.split('base64,')[1]
            let base64data = split ? split : tokenURI

            const output = Buffer.from(base64data, 'base64').toString('utf-8')

            console.log( output )
            
            this.encodedImageSVG = JSON.parse(output) .image 

             console.log( this.encodedImageSVG)


            return output 
          },


          async mint( ){


            if(!this.signedInToWeb3){

              this.web3Plug.connectWeb3()
              return 

            }

            let userAddress = this.web3Plug.getActiveAccountAddress()
             let amt =  ( this.mintAmount ).toString()

             if( parseInt(amt) > 10  ){
               this.errorMessage = 'You may only mint up to 10 at once. '
               return 
             }
             this.errorMessage = null 

             let ethValue = parseInt(amt) * 2 * 10000000000000000 ; 

            let contractData = await this.web3Plug.getContractDataForActiveNetwork()

            const nftContract = this.web3Plug.getCustomContract( StatsNFTABI, contractData.statsnft.address  )

            this.totalSupply = await nftContract.methods.claim( amt ).send({from: userAddress, value: ethValue })
  

             
          },  
        

         
 

  }
}
</script>
