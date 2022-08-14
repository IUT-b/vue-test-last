<template>
  <div class="hello">
    <div>
      <li>Counter:{{n}}</li>
    </div>
    <button style="submit" v-on:click="add">追加</button>
    <button style="submit" @click="reset">リセット</button>
    <br>
    <table>
        <tbody>
            <tr v-for="counter in counters" :key="counter.text">
                <td><input v-model="counter.item"></td>
                <td><input v-model="counter.number"></td>
            </tr>
            <tr>
                <td><input v-model="counter.item"></td>
                <td><input v-model="counter.number"></td>
            </tr>
        </tbody>
    </table>

    <table>
        <thead>
            <tr>
                <th>編集</th>
                <th>Dessert (100g serving)</th>
                <th>Calories</th>
                <th>Fat (g)</th>
                <th>Carbs (g)</th>
                <th>Protein (g)</th>
                <th>Iron (%)</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="item in items" :key="item.id">
                <td><button v-on:click="edit(item)">編集</button><button v-on:click="del(item.id)">削除</button></td>

                    <div id="overlay" v-show="showContent">
                      <!-- <div id="overlay" v-show="showContent"> -->
                    <div id="content">
                    <p>name</p>
                    <input v-model="NAME">
                    <p>calories</p>
                    <input v-model="CAL">
                    <br>
                    <button v-on:click="closeModal(ID)">編集完了</button>
                    </div>
                    </div>

                <td>{{item.name}}</td>
                <td>{{item.cal}}</td>
                <td>{{item.fat}}</td>
                <td>{{item.car}}</td>
                <td>{{item.pro}}</td>
                <td>{{item.iro}}%</td>
            </tr>
        </tbody>
    </table>
  </div>
</template>

<script>
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import {
  getFirestore, collection, doc, getDocs, updateDoc, deleteDoc, onSnapshot//,query, where, orderBy, getDoc, setDoc
} from "firebase/firestore"
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "AIzaSyBDspN5wYMpLLptflRAThOKDOyXOJ7efFs",
  authDomain: "vue-test20220812.firebaseapp.com",
  projectId: "vue-test20220812",
  storageBucket: "vue-test20220812.appspot.com",
  messagingSenderId: "635392134107",
  appId: "1:635392134107:web:9fd86c1f09e60c9047f5fe"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const db=getFirestore(app)
// const db=getFirestore()




export default {
  name: 'HelloWorld',
  data: () => ({
    counters:[],
    counter: {
      item: "",
      number: "",
    },
    n:0,
    items:[],
    data:[],
    ID:"",
    NAME:"",
    CAL:"",

    showContent: false

  }),

////////////////////////////////////////
  mounted: function () {
    this.readData();
  },
////////////////////////////////////////


created(){
this.listItems()
},


  methods:{



listItems(){
const q=collection(db,"vue-test")
onSnapshot(q,(snapShot)=>{
this.items=[]
snapShot.forEach((doc)=>{
  const data=doc.data()
  data.id=doc.id
  this.items.push(data)
})
}
)
},




  /////////////////////////////////////////////////
    async readData(){
      const querySnapshot = await getDocs(collection(db, "vue-test"));
      ////////////////////////////////
      // const data=doc.data()
      // data.id=doc.id
      /////////////////////////////////
      querySnapshot.forEach((doc) => {
        // console.log(`${doc.id} => ${doc.data()}`);
        // console.log(`${doc.id} => ${doc.data().name}`);
        // this.data.push(doc.data().name)
        this.data.push(doc.data())
      });
    },
  /////////////////////////////////////////////////



    add:function(){
      this.counters.push(this.counter)
      this.n=Number(this.n)+Number(this.counter.number)
      this.counter={
      item: "",
      number: ""
    }
    },

    reset:function(){
      this.counters=[]
      this.n=0
    },

    edit: function(item){
      this.showContent = true
//       // this.dessert=item
      // const docRef = doc(db, "vue-test", docId);
      // const docSnap = getDoc(docRef);
// this.test=docSnap
this.ID=item.id
this.NAME=item.name
this.CAL=item.cal
//  console.log(docId)
// const data={dessert:this.item}
    // const data={test:this.test}

    },

    closeModal: function(docId){
      // // this.desserts.push(this.dessert)
      // const data={dessert:this.dessert}
      // const path="vue-test"
      // const docPath=doc(collection(db,path))
      // setDoc(docPath,data)

      // // const cityRef = doc(db, 'vue-test', 'Frozen Yogurt')
      // // setDoc(cityRef, { merge: true })

      // console.log(item.id)
      // console.log(docId)
      const Ref = doc(db, "vue-test", docId);
        updateDoc(Ref, {
          // name: item.name,
          // cal: item.cal,
          name: this.NAME,
          cal: this.CAL
        })
      .catch(err=>{
        console.log("err:",err)
      })




      this.showContent = false
    },

    del: function(docId){
      deleteDoc(doc(db, "vue-test", docId))
    },



  }
/*   props: {
    msg: String
  } */
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* h3 {
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
} */




#overlay{
  /*要素を重ねた時の順番*/
  z-index:1;

  /*画面全体を覆う設定*/
  position:fixed;
  top:0;
  left:0;
  width:100%;
  height:100%;
  background-color:rgba(0,0,0,0.5);

  /*画面の中央に要素を表示させる設定*/
  display: flex;
  align-items: center;
  justify-content: center;

}

#content{
  z-index:2;
  width:50%;
  padding: 1em;
  background:#fff;
}






</style>
