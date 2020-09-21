<template>
  <div class="content" role="main">
    <div style="text-align:center">
      <h1>{{ title }} app is running!</h1>
      <h2>Advanced Printing from Javascript</h2>
      <fieldset>
        <legend>Client Printer:</legend>
        <label class="checkbox">
          <input type="checkbox" id="useDefaultPrinter" 
            v-model="print2default" />
          <strong>Print to Default printer</strong>
        </label>
        <p>or...</p>
        <Printers :printer_list="printers" v-on:change="onPrinterChange" />
      </fieldset>
      <br />
      <br />
      <button v-on:click="doPrintZPL">Print ZPL commands (Zebra printer required)...</button>&nbsp;
      <strong>OR</strong>&nbsp;
      <button v-on:click="doPrintPDF">Print PDF...</button>
    </div>
  </div>
</template>

<script>
import Printers from "./Printers";
import JSPM from "jsprintmanager";
// import Vue from 'vue';

export default {
  name: "Home",
  props: ["title"],
  components: {
    Printers
  },
  data: function () {
    return {
      print2default: false,
      printers: [],
      selected_printer: ''
    }
  },
  methods: {
    doPrintZPL: function(){
      if(this.selected_printer === '' && !this.print2default) {
        alert("You must select a printer");
        return;
      }
      let cpj = new JSPM.ClientPrintJob();
      if ( this.print2default ) {
        cpj.clientPrinter = new JSPM.DefaultPrinter();
      } else {
        cpj.clientPrinter = new JSPM.InstalledPrinter(this.selected_printer);
      }
      var cmds =  "^XA";
      cmds += "^FO20,30^GB750,1100,4^FS";
      cmds += "^FO20,30^GB750,200,4^FS";
      cmds += "^FO20,30^GB750,400,4^FS";
      cmds += "^FO20,30^GB750,700,4^FS";
      cmds += "^FO20,226^GB325,204,4^FS";
      cmds += "^FO30,40^ADN,36,20^FDShip to:^FS";
      cmds += "^FO30,260^ADN,18,10^FDPart number #^FS";
      cmds += "^FO360,260^ADN,18,10^FDDescription:^FS";
      cmds += "^FO30,750^ADN,36,20^FDFrom:^FS";
      cmds += "^FO150,125^ADN,36,20^FDAcme Printing^FS";
      cmds += "^FO60,330^ADN,36,20^FD14042^FS";
      cmds += "^FO400,330^ADN,36,20^FDScrew^FS";
      cmds += "^FO70,480^BY4^B3N,,200^FD12345678^FS";
      cmds += "^FO150,800^ADN,36,20^FDMacks Fabricating^FS";
      cmds += "^XZ";
      cpj.printerCommands = cmds;
      cpj.sendToClient();
    },
    doPrintPDF: function(){
      if(this.selected_printer === '' && !this.print2default) {
        alert("You must select a printer");
        return;
      }
      let cpj = new JSPM.ClientPrintJob();
      if (this.print2default) {
        cpj.clientPrinter = new JSPM.DefaultPrinter();
      } else {
        cpj.clientPrinter = new JSPM.InstalledPrinter(this.selected_printer);
      }
      var my_file = new JSPM.PrintFilePDF(
        'https://neodynamic.com/temp/LoremIpsum.pdf', 
        JSPM.FileSourceType.URL, 
        'MyFile.pdf', 
        1);
      cpj.files.push(my_file);
      cpj.sendToClient();
    }, 
    onPrinterChange: function(value) {
      this.selected_printer = value;
      console.info("Selected printer:", value);
    },
    onInit: function() {
      var _this = this;
      JSPM.JSPrintManager.auto_reconnect = true;
      JSPM.JSPrintManager.start();
      JSPM.JSPrintManager.WS.onStatusChanged = function () {
        _this.getPrinters().then((p)=>{
          _this.printers = p;
          _this.$nextTick();
        });
      };
    },
    getPrinters: function() {   
      return new Promise((ok, err) => {
        let printers = [];
        if(JSPM.JSPrintManager.websocket_status == JSPM.WSStatus.Open) {
          JSPM.JSPrintManager.getPrinters().then(function (myPrinters) {
            printers = myPrinters;
            console.log(printers);
            ok(printers);
          }).catch((e)=>err(e));
        } else { console.warn("JSPM WS not open"); ok(printers); }
      });       
    }
  },
  mounted: function() {
    this.onInit();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
