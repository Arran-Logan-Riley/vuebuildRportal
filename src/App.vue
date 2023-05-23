<template>
  <div>
    <h1>[Vue FireBase Portal]</h1>
    <button class="button" v-if="showLoginButton" @click="handleGoogleLogin">Login With Google</button>
    <button class="button" v-if="showSignOutButton" @click="handleGoogleSignOut">Sign Out</button>
    <div>
      <label class="welcomeMessage" v-if="showUserLoginName">Welcome: {{ userDisplayName }}</label>
    </div>
    <section>
      <ul>
        <li class="container" v-for="item in fireStoreData" :key="item.id">
          <div class="message">{{ item.projectName }}
            <div>Start Date:{{ item.startDate }}</div>
            <div>End Date:{{ item.endDate }}</div>
            <div>Material Required: {{ item.material }} Number: {{ item.materialNum }}</div>
            <div>Message ID: {{ item.msgId }}</div>
          </div>
        </li>
      </ul>
    </section>

  </div>
</template>

<script>
import { ref } from 'vue';
import { initializeApp } from 'firebase/app';
import { getFirestore, collection, getDocs } from 'firebase/firestore';
import { getAuth, GoogleAuthProvider, signInWithPopup, signOut as firebaseSignout } from 'firebase/auth';

import firebaseConfig from './config.json'

const firebaseApp = initializeApp(firebaseConfig);
const auth = getAuth(firebaseApp);
const firestore = getFirestore(firebaseApp);

const provider = new GoogleAuthProvider();

export default {
  name: 'App',
  setup() {
    //create a firestore data refrence to be used in the query method
    const fireStoreData = ref([]);
    const showLoginButton = ref(true);
    const showSignOutButton = ref(false);
    const showUserLoginName = ref(false);
    const userDisplayName = ref(null);
    const user = ref(null);
    

    const handleGoogleLogin = async () => {
      try {
        const result = await signInWithPopup(auth, provider);
        const { user } = result;
        console.log('Logged in user:', user);
        queryFirestore();
        showLoginButton.value = false;
        showSignOutButton.value = true;
        showUserLoginName.value = true;
        // Update the user value with the logged-in user
        user.value = user;
        userDisplayName.value = user.displayName;
      } catch (error) {
        console.error('Login error:', error);
      }
    };
    const handleGoogleSignOut = async () => {
      try {
        await firebaseSignout(auth)
        showLoginButton.value = true;
        showSignOutButton.value = false;
        showUserLoginName.value = false;
        user.value = null;
        fireStoreData.value = null;
        console.log(user);
      } catch (error) {
        console.log(error);
      }
    }
    //Method to call my firestore DB theen in a data snapshot, save the data to a data array.
    const queryFirestore = async () => {
      try {
        const collectionRef = collection(firestore, 'messages');
        const querySnapshot = await getDocs(collectionRef);
        const data = [];
        querySnapshot.forEach((doc) => {
          console.log(doc.data());
          data.push({
            id: doc.id,
            ...doc.data(),
          });
        });
        fireStoreData.value = data;
      } catch (error) {
        console.log(error);
      }
    };

    return {
      fireStoreData,
      handleGoogleLogin,
      handleGoogleSignOut,
      queryFirestore,
      showLoginButton,
      showSignOutButton,
      showUserLoginName,
      userDisplayName,
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

}

body {
  background-color: #0b1a26;
}

.message {
  align-items: center;
  max-width: 400px;
  padding-left: 30px;
  padding-right: 30px;
  padding-bottom: 5px;
  padding-top: 2px;
  margin: 10px;
  border-radius: 25px;
  border-color: white;
  border: 2px;
  color: white;
  border-style: solid;
  background: #0b93f6;
}

.button {
  background: none;
  margin: 10px;
  height: 40px;
  border: 2px solid #ffd52d;
  border-radius: 50px;
  box-sizing: border-box;
  font-size: 26px;
  color: #ffd52d;
  outline: none;
  transition: .5s;
}

.button:hover {
  background: #3b3640;
  border-radius: 10px;
}

.container {
  display: flex;
  flex-wrap: wrap;
}

ul,
li {
  text-align: left;
  list-style: none;
}

main {
  padding: 10px;
  height: 80vh;
  margin: 10vh 0 10vh;
  overflow-y: scroll;
  display: flex;
  flex-direction: column;
}

section {
  flex-direction: column;
  justify-content: center;
  min-height: 100vh;
  background-color: rgb(40, 37, 53);
}
</style>
