<template>
  <div>
<div>
  <b-button v-b-modal.modal-1 @click="getSelectedRows()">Get Selected Row(s)</b-button>

  <b-modal id="modal-1" title="Stock">
   
    <EditForm :stock-data="this.stockselection" />
   
      <template v-slot:modal-footer="{ cancel }">
        <b-button @click="cancel()">Cancel</b-button>
      </template>

  </b-modal>
</div>

    <ag-grid-vue style="width: 1400px; height: 800px;"
        class="ag-theme-alpine"
        :columnDefs="columnDefs"
        :rowData="rowData"
        rowSelection="multiple"
        @grid-ready="onGridReady">
    </ag-grid-vue>

  </div>
</template>

<script>
    import { AgGridVue } from "ag-grid-vue";
    import EditForm from '@/components/EditForm';

    export default {
        name: 'App',
        data() {
            return {
                columnDefs: null,
                rowData: null,
                gridApi: null,
                columnApi: null,
  
                stockselection: [
                  {
                    id: 0,
                    sku: "",
                    store: "",
                    style: "",
                    colour: "",
                    qty: "",
                    size: ""
                  }
                 ],         
            }
        },
        components: {
            AgGridVue,
            EditForm
        },
        methods: {
            onGridReady(params) {
                this.gridApi = params.api;
                this.columnApi = params.columnApi;
            },
            getSelectedRows() {                
       
                if(this.gridApi.getSelectedNodes().length===0)
                {                 
                    this.gridApi.selectAllFiltered();                                  
                }
                const selectedNodes = this.gridApi.getSelectedNodes();
                const selectedData = selectedNodes.map( node => node.data );

                selectedData.sort(function(a, b){
                    var a1= a.sizeCode.toLowerCase(), b1= b.sizeCode.toLowerCase();        
                    if(a1== b1) return 0;
                    return a1> b1? 1: -1;
                });
             
                selectedData.sort(function(a, b){
                    var a1= a.store.toLowerCase(), b1= b.store.toLowerCase();        
                    if(a1== b1) return 0;
                    return a1> b1? 1: -1;
                });
               
              const styleOptionList = [];
          
              var jsonMsg = {};
              var styleOptions = []
              jsonMsg.styleOptions = styleOptions;

              for(var i=0;i<selectedNodes.length;i++)
              {

                if(!_isContains(jsonMsg, "name", selectedData[i].styleOption))
                {      
                  // add new style option                      
                  jsonMsg.styleOptions.push({name: selectedData[i].styleOption});
                  
                  // add new size option
                 var sizes = [];     
                 jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes = sizes;
                 jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes.push({name: selectedData[i].sizeCode});

                  // add new store
                  var stores = []
                 jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes[jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes.length-1].stores = stores;
                 jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes[jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes.length-1].stores
                 jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes[jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes.length-1].stores.push({name: selectedData[i].store, qty: selectedData[0].allocatedUnits});

                }else{
                  // get current style node
                  var currentStyleNode = _getNode(jsonMsg.styleOptions,selectedData[i].styleOption);
                  
                  //add new size code if it does not exist
                 if(!_isContains(currentStyleNode, "name", selectedData[i].sizeCode))
                 {
                   var stores = []
                   currentStyleNode.sizes.push({name: selectedData[i].sizeCode, stores: stores});
                 }

                   // get current size node
                   var currentSizeNode = _getNode(currentStyleNode.sizes,selectedData[i].sizeCode);

                   //add new store name if it does not exist
                  if(!_isContains(currentSizeNode, "name", selectedData[i].store))
                  {
                    currentSizeNode.stores.push({name: selectedData[i].store, qty: selectedData[i].allocatedUnits});
                  }

                }
           
          }

          //alert(JSON.stringify(jsonMsg));
          this.stockselection = jsonMsg;

          },

        },
        beforeMount() {
            this.columnDefs = [
              {field: 'partnerStockId', sortable: true, filter: true, checkboxSelection: true},
              { field: 'productSKU', sortable: true, filter: true },
              { field: 'store', sortable: true, filter: true },
              { field: 'styleOption', sortable: true, filter: true },
              { field: 'colourDesc', sortable: true, filter: true },
              { field: 'sizeCode', sortable: true, filter: true },
              { field: 'allocatedUnits', sortable: true, editable: true }
                            
            ];

        // fetch('https://dev-ist-api.bodendev.com/api/ist/v1/partner-stock')
        //   .then(result => result.json())
        //   .then(rowData => this.rowData = rowData);
        // }

        fetch('https://localhost:5001/api/ist/v1/partner-stock')
          .then(result => result.json())
          .then(rowData => this.rowData = rowData);
        }
    }


   function _isContains(json, keyname, value) {
    return Object.keys(json).some(key => {
            return typeof json[key] === 'object' ? 
            _isContains(json[key], keyname, value) : key === keyname && json[key] === value;
        });
  }

  function _getNode(json,keyname)
  {

    for(var i=0;i<json.length;i++)
    {
      if(json[i].name===keyname)
      {
        return json[i];
      }
    }
    
  }

</script>


<style lang="scss">
  @import "../node_modules/ag-grid-community/dist/styles/ag-grid.css";
  @import "../node_modules/ag-grid-community/dist/styles/ag-theme-alpine.css";
</style>
