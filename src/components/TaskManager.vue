<template>
  <div>
    
  

      <div v-for="person, index, key in employee" :key="key">
        
    <input  v-if="index != 'id'" type="text"  :placeholder="index" v-model="employee[index]" >
  </div>
  <button @click="clear">clear</button>
  <button @click="add(employee)">Add data </button>
  <div class="container">
    <label for="example-datepicker">Choose a date</label>
    <b-form-datepicker id="example-datepicker" v-model="employee.date"></b-form-datepicker>
    
  </div>
  <div class="container">
    <b-form-timepicker v-model="employee.time" locale="en"></b-form-timepicker>
    
  </div>
  <div class="container-md">
    <div class="row">
    <div class="col order-1">
  <ul v-for="(employee, index, key) in employees" :key="key" class="list-group">
    <li   v-if="employee" class="list-group-item d-flex justify-content-between align-items-center">
      
      
      <input v-if="employee.edit" type="text" v-model="employee.name">
      <span v-else>Имя: {{employee.name}}</span> <br/> 
       <input v-if="employee.edit" type="text" v-model="employee.comment">
      <span v-else>Комментарий: {{employee.comment}}</span>  <br/>
       <input v-if="employee.edit" type="text" v-model="employee.status">
      <span v-else>Статус: {{employee.status}}</span> <br/> 
       <input v-if="employee.edit" type="text" v-model="employee.date">
      <span v-else>Дата: {{employee.date}}</span>  <br/>
      <input v-if="employee.edit" type="text" v-model="employee.time">
      <span v-else>Время: {{employee.time}}</span>  <br/>       
      <button v-if="!employee.edit" @click="employee.edit = !employee.edit">&#9998;</button> 
      <button v-if="employee.edit" @click="edit(employee, index)">&#9998;</button>
      <button @click="remove(employee.id, index)" >&#9747;</button>
    </li>
  </ul>
</div>
</div>
  </div>
  
  </div>
</template>

<script>
const employeeData = [
];
var db;
var request = window.indexedDB.open("newDatabase", 4);

request.onerror = function() {
  console.log("error: ");
};

request.onsuccess = function() {
  db = request.result;
  console.log("success: "+ db);
};

request.onupgradeneeded = function(event) {
  var db = event.target.result;
  var objectStore = db.createObjectStore("employee", {keyPath: "id"});

  for (var i in employeeData) {
    objectStore.add(employeeData[i]);
  }
}

export default {
   data(){
    return {
      
      employee:{
        id: null,
        name: null,
        comment: null,
        status: null,
        date: '',
        time: ''
      },
      employees: [],
      retryCount: 0,
      

    }
  },
  mounted(){
    this.readAll()
  },

  methods:{
    add(employee) {
      var vm = this
      new Promise(() => {
        var request = db.transaction(["employee"], "readwrite")
        .objectStore("employee").add({ 
          id: employee.id != null ? employee.id : vm.employees.length++ ,
          name: employee.name, 
          comment: employee.comment, 
          status:employee.status,
          date: employee.date,
          time: employee.time
        })
        request.onsuccess = function() {
          vm.employees.push({ 
            edit: false,
            id: employee.id != null ? vm.employee.id : vm.employees.length++ ,
            name: employee.name, 
            comment: employee.comment, 
            status: employee.status,
            date: employee.date,
            time: employee.time
          })
        }
        request.onerror = function() {
          alert("Unable to add data ");
        }
      }); 
    },
    clear() {
      var vm = this
     db.transaction(["employee"], "readwrite")
      .objectStore("employee")
      .clear()
      vm.employees = []
    },
    edit(employee, index) {

      var vm = this 
      new Promise((resolve) => {
        vm.remove(employee.id, index)
        resolve(vm.add(employee)); 
      });

    },
    readAll() {
      var vm = this 
      try {
        vm.employees = []
        var objectStore = db.transaction("employee").objectStore("employee");
        if(objectStore){
          objectStore.openCursor().onsuccess = function(event) {
            var cursor = event.target.result;

            if (cursor) {
              if(vm.employees){
                vm.employees.push({ edit: false,  id: cursor.key , name: cursor.value.name , comment:cursor.value.comment, status:cursor.value.status, date:cursor.value.date, time:cursor.value.time});

              }
              cursor.continue();

            }
          };
        }
      }catch(e){
        vm.retryDisp()
      }
    },
    remove(id, index) {
      var vm = this 
      db.transaction(["employee"], "readwrite")
      .objectStore("employee")
      .delete(id);
      vm.employees.splice(index, 1)
    },
    retryDisp() {
      var vm = this
      if( ++vm.retryCount > 5 ) {
        console.log('Cannot open the database after 5 retries');
        vm.readAll();
      }
      setTimeout(function(){ vm.readAll() }, 100);
    }
  }
}
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
