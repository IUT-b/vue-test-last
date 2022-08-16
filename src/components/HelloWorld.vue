<template>
  <div>
    <v-app>
      <!-- カウンター機能 -->
      <v-container fluid>
        <!-- カウンター結果表示 -->
        <v-row no-gutters>
          <v-col> Counter:{{ calTotal }} </v-col>
        </v-row>

        <!-- カウンターボタン -->
        <v-row no-gutters>
          <v-col>
            <v-btn @click="add">追加</v-btn>
            <v-btn @click="reset">リセット</v-btn>
          </v-col>
        </v-row>

        <!-- 入力済のフォーム表示 -->
        <v-row no-gutters v-for="dessert in desserts" :key="dessert.name">
          <v-col>
            <v-text-field outlined dense v-model="dessert.name" />
          </v-col>
          <v-col>
            <v-text-field type="number" outlined dense v-model="dessert.cal" />
          </v-col>
        </v-row>

        <!-- 入力フォーム -->
        <v-row no-gutters>
          <v-col>
            <v-text-field outlined dense v-model="dessert.name" />
          </v-col>
          <v-col>
            <v-text-field type="number" outlined dense v-model="dessert.cal" />
          </v-col>
        </v-row>
      </v-container>

      <!-- データベース機能(Firestoreのデータベース使用) -->
      <v-container fluid>
        <!-- データベース表 -->
        <v-row>
          <v-col>
            <v-data-table
              :headers="headers"
              :items="items"
              sort-by="cal"
              class="elevation-1"
            >
              <!-- データ(Firestoreのドキュメント)の編集・削除ボタン追加 -->
              <template v-slot:[`item.edit`]="{ item }">
                <v-btn depressed outlined color="blue" @click="editItem(item)"
                  >編集</v-btn
                >
                &thinsp;
                <v-btn
                  depressed
                  outlined
                  color="red"
                  @click="deleteItem(item.id)"
                  >削除</v-btn
                >
              </template>
            </v-data-table>
          </v-col>
        </v-row>

        <!-- 編集ボタン押下時のダイアログ -->
        <v-row justify="center">
          <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
              <v-card-text>
                <v-container>
                  <!-- データ(Firestoreのドキュメントのname)の変更 -->
                  <v-row no-gutters>
                    <v-col>
                      <div style="font-weight: bold">name</div>
                    </v-col>
                  </v-row>
                  <v-row no-gutters>
                    <v-col>
                      <v-textarea outlined v-model="selectedDoc.name" />
                    </v-col>
                  </v-row>

                  <!-- データ(Firestoreのドキュメントのcal)の変更 -->
                  <v-row no-gutters>
                    <v-col>
                      <div style="font-weight: bold">calories</div>
                    </v-col>
                  </v-row>
                  <v-row no-gutters>
                    <v-col>
                      <v-text-field
                        type="number"
                        outlined
                        v-model="selectedDoc.cal"
                      />
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <!-- 変更の保存 -->
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  depressed
                  outlined
                  color="blue darken-1"
                  @click="saveItem()"
                  >編集完了
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-row>
      </v-container>
    </v-app>
  </div>
</template>

<script>
// Firestoreインポート
import { initializeApp } from "firebase/app";
import {
  getFirestore,
  collection,
  doc,
  updateDoc,
  deleteDoc,
  onSnapshot,
} from "firebase/firestore";

const firebaseConfig = {
  apiKey: "AIzaSyBDspN5wYMpLLptflRAThOKDOyXOJ7efFs",
  authDomain: "vue-test20220812.firebaseapp.com",
  projectId: "vue-test20220812",
  storageBucket: "vue-test20220812.appspot.com",
  messagingSenderId: "635392134107",
  appId: "1:635392134107:web:9fd86c1f09e60c9047f5fe",
};

// Firestore初期化
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

export default {
  data: () => ({
    // カウンター機能用
    desserts: [], // カウンターの全データ格納
    dessert: {
      name: "",
      cal: "",
    }, // カウンターの各データ格納
    calTotal: 0, // カウント数：初期設定=0

    // データベース機能用
    data: [], // Firestoreのコレクション格納(ドキュメントIDなし)
    items: [], // Firestoreのコレクション格納(ドキュメントID追記)
    selectedDoc: {
      id: "",
      name: "",
      cal: "",
    }, // Firestoreから選択したドキュメント格納

    dialog: false, // ダイアログ表示：初期設定=非表示

    headers: [
      { text: "編集", sortable: false, value: "edit" },
      {
        text: "Dessert (100g serving)",
        align: "start",
        sortable: false,
        value: "name",
      },
      { text: "Calories", value: "cal" },
      { text: "Fat (g)", value: "fat" },
      { text: "Carbs (g)", value: "car" },
      { text: "Protein (g)", value: "pro" },
      { text: "Iron (%)", sortable: false, value: "iro" },
    ], // データベース表のヘッダー
  }),

  // Firestoreのコレクション読み込み
  mounted: function () {
    this.readData();
  },

  // Firestoreのコレクション読み込みデータにドキュメントIDを追記
  created() {
    this.listItems();
  },

  methods: {
    // カウンター機能用
    // カウント
    add() {
      this.desserts.push(this.dessert);
      this.calTotal = Number(this.calTotal) + Number(this.dessert.cal);
      this.dessert = {
        name: "",
        cal: 0,
      };
    },

    // カウントのリセット
    reset() {
      this.desserts = [];
      this.dessert = {
        name: "",
        cal: "",
      };
      this.calTotal = 0;
    },

    // データベース機能用
    // Firestoreのコレクション読み込み
    async readData() {
      const q = collection(db, "vue-test");
      onSnapshot(q, (snapShot) => {
        snapShot.forEach((doc) => {
          this.data.push(doc.data());
        });
      });
    },

    // Firestoreのコレクション読み込みデータにドキュメントIDを追記
    listItems() {
      const q = collection(db, "vue-test");
      onSnapshot(q, (snapShot) => {
        this.items = [];
        snapShot.forEach((doc) => {
          const data = doc.data();
          data.id = doc.id;
          this.items.push(data);
        });
      });
    },

    // Firestoreのドキュメント編集
    editItem(item) {
      this.dialog = true;
      this.selectedDoc = {
        id: item.id,
        name: item.name,
        cal: item.cal,
      };
    },

    // Firestoreのドキュメント保存
    saveItem() {
      const ref = doc(db, "vue-test", this.selectedDoc.id);
      updateDoc(ref, {
        name: this.selectedDoc.name,
        cal: Number(this.selectedDoc.cal),
      }).catch((err) => {
        console.log("err:", err);
      });
      this.dialog = false;
    },

    // Firestoreのドキュメント削除
    deleteItem(docId) {
      deleteDoc(doc(db, "vue-test", docId));
    },
  },
};
</script>

<style scoped>
</style>