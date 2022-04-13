<template>
  <div class="bg-gray-600 shadow-sm text-center text-white sticky top-0 z-50">
      <div class="px-2 py-2 sm:px-6">
        <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="page > 1 ? page-- : 1">Previous</button> &nbsp;
        <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="page < numPages ? page++ : 1">Next</button>&nbsp;&nbsp;
        <span class="mt-1 max-w-2xl text-md">Page : {{page}} / {{ numPages ? numPages : '∞' }}</span>
      </div>
  </div>

  
  <div id = "pdfvuer" class = "relative">
      <div class="bg-grey shadow overflow-hidden sm:rounded-lg">
      <div class="border-t border-gray-200">
        <pdf :src="pdfdata" v-for="i in numPages" :key="i" :id="i" :page="i"
            style="width:60%;margin:20px auto;"
            @link-clicked="handle_pdf_link">
        </pdf>
      </div>
    </div>
  </div>
</template>

<script>
import pdfvuer from 'pdfvuer'
import 'pdfjs-dist/build/pdf.worker.entry' 

export default {
  components : {
    pdf : pdfvuer
  },
  data () {
    return {
      page : 1,
      numPages : 0,
      pdfdata : undefined,
      errors: [],
    }
  },
  mounted() {
    this.getPdf()
  },
  watch: {
    show:function (s){
      if(s){
        this.getPdf();
      }
    },
    page: function (p){
        if( window.pageYOffset <= this.findPos(document.getElementById(p)) || ( document.getElementById(p+1) && window.pageYOffset >= this.findPos(document.getElementById(p+1)) )) {
        document.getElementById(p).scrollIntoView();
      }
    }
  },
  methods: {
    handle_pdf_link: function (params) {
      var page = document.getElementById(String(params.pageNumber));
      page.scrollIntoView();
    },
    getPdf () {
      var self = this;
      self.pdfdata = pdfvuer.createLoadingTask('https://raw.githubusercontent.com/mozilla/pdf.js/ba2edeae/web/compressed.tracemonkey-pldi-09.pdf');
      self.pdfdata.then(pdf => {
        self.numPages = pdf.numPages;
        window.onscroll = function() { 
          changePage() 
        }
        function changePage () {
          var i = 1, count = Number(pdf.numPages);
          do {
            if(window.pageYOffset >= self.findPos(document.getElementById(i)) && 
                window.pageYOffset <= self.findPos(document.getElementById(i+1))) {
              self.page = i
            }
            i++
          } while ( i < count)
          if (window.pageYOffset >= self.findPos(document.getElementById(i))) {
            self.page = i
          }
        }
      });
    },
    findPos(obj) {
      return obj.offsetTop;
    }
  }
};
</script>

<style>
body {
  background: lightgrey !important;
}
</style>
