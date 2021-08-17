<template>
  <div>
    <div>
      <div class="header-button">
      <select class="form-control" @change="changeJobTitle($event)">
        <option value="" selected disabled>BI Selection</option>
        <option v-for="biSelection in biSelections" :value="biSelection" :key="biSelection">{{ biSelection }}</option>
      </select>
        <b-button variant="primary" v-b-modal.modal-1 @click="getSelectedRows()">Allocation</b-button>
        <b-button variant="success" v-b-modal.modal-2 @click="getSelectedRows()">Replenishment</b-button>
        <b-button variant="danger" >Approve</b-button>
        <b-button class="clear" @click="clearSelectedRows()">Clear</b-button>

      </div>
      <b-modal id="modal-1" title="Stock Allocation">      
        <EditForm :stock-data="this.stockselection" />      
          <template v-slot:modal-footer="{ cancel }">
            <b-button @click="cancel()">Cancel</b-button>
          </template>
      </b-modal>

      <b-modal id="modal-2" title="Stock Replenishment">      
        <ReplenishmentForm :stock-data="this.stockselection" />      
          <template v-slot:modal-footer="{ cancel }">
            <b-button @click="cancel()">Cancel</b-button>
          </template>
      </b-modal>

    </div>

    <ag-grid-vue style="width: 1860px; height: 800px;"
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
    import ReplenishmentForm from '@/components/ReplenishmentForm';
    
    export default {
        name: 'App',
        
        data() {
            return {
                columnDefs: null,
                rowData: null,
                gridApi: null,
                columnApi: null,
                biSelections: null,
  
                stockselection: [
                  {
                    id: 0,
                    sku: "",
                    store: "",
                    style: "",
                    colour: "",
                    unit: "",
                    qty: "",
                    size: "",
                    checked: true
                  }
                 ],                       
            }
        },
        components: {
            AgGridVue,
            EditForm,
            ReplenishmentForm
        },
        methods: {
            changeJobTitle (event) {

              var selection = event.target.options[event.target.options.selectedIndex].text;
              var url = 'https://localhost:5001/api/ist/v1/partner-stock-option/selection='+selection

              fetch(url)
                .then(result => result.json())
                .then(rowData => this.rowData = rowData);
            },
            onGridReady(params) {
                this.gridApi = params.api;
                this.columnApi = params.columnApi;
            },
            clearSelectedRows()
            {
              this.gridApi.deselectAll();
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
                 jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes[jsonMsg.styleOptions[jsonMsg.styleOptions.length-1].sizes.length-1].stores.push({name: selectedData[i].store, unit: selectedData[0].allocatedUnits, qty: selectedData[0].qty});

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
                    currentSizeNode.stores.push({name: selectedData[i].store, unit: selectedData[i].allocatedUnits, qty: selectedData[i].qty});
                  }

                }
           
          }

          //alert(JSON.stringify(jsonMsg));
          this.stockselection = jsonMsg;

          },

        },
        beforeMount() {
            this.columnDefs = [
              {field: 'partnerStockId', sortable: true, filter: true, checkboxSelection: true, headerName: "Id", width: 160},   
              { field: 'division', sortable: true, filter: true, headerName: "Div", width: 160 },           
              { field: 'segment', sortable: true, filter: true, headerName: "Seg", width: 160 },                
              { field: 'department', sortable: true, filter: true, headerName: "Dpt", width: 160 }, 
              { field: 'category', sortable: true, filter: true, headerName: "Cat", width: 160 },   
              { field: 'productSKU', sortable: true, filter: true, headerName: "SKU", width: 160 },  
              { field: 'store', sortable: true, filter: true, headerName: "Store", width: 160 },  
              { field: 'styleOption', sortable: true, filter: true, headerName: "Opt", width: 160 },  
              { field: 'colourDesc', sortable: true, filter: true, headerName: "Col", width: 160 },  
              { field: 'sizeCode', sortable: true, filter: true, headerName: "Size", width: 160 },  
              { field: 'allocatedUnits', sortable: true, headerName: "Unit", width: 160 },
              { field: 'qty', sortable: true, editable: true, headerName: "Qty", width: 160 }                            
            ];

        // fetch('https://dev-ist-api.bodendev.com/api/ist/v1/partner-stock')
        //   .then(result => result.json())
        //   .then(rowData => this.rowData = rowData);
        // }

        fetch('https://localhost:5001/api/ist/v1/bi-selection?partner=jl')
          .then(result => result.json())
          .then(biSelections => this.biSelections = biSelections);
        }

        // fetch('https://localhost:5001/api/ist/v1/partner-stock')
        //   .then(result => result.json())
        //   .then(rowData => this.rowData = rowData);
        // }
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
  
  .header-button .btn{
    margin: 5px;
  }

   .header-button .clear{
    float: right;
  }
  
</style>
