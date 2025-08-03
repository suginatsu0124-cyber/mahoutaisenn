// src/firebase.js

import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';
import { getFirestore, doc, getDoc, setDoc, updateDoc, serverTimestamp } from 'firebase/firestore';

// Firebase設定のグローバル変数
// この部分には、ご自身のFirebaseプロジェクトの設定情報を貼り付けてください
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

// Firebaseサービスの初期化
const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);

/**
 * 匿名認証を行い、ユーザーデータを取得または作成する
 * @param {string} customToken - サーバーから提供されるカスタムトークン（任意）
 * @returns {Promise<Object>} ユーザーデータ
 */
export const signInAndFetchUserData = async (customToken = null) => {
  try {
    if (customToken) {
      await signInWithCustomToken(auth, customToken);
    } else {
      await signInAnonymously(auth);
    }
    
    const user = auth.currentUser;
    if (!user) throw new Error("ユーザーが認証されていません。");

    const userRef = doc(db, 'users', user.uid);
    const userSnap = await getDoc(userRef);

    if (!userSnap.exists()) {
      const initialData = {
        currency: 100,
        unlockedCharacters: ['タメゾー'],
        wins: 0,
        createdAt: serverTimestamp(),
      };
      await setDoc(userRef, initialData);
      return initialData;
    } else {
      return userSnap.data();
    }
  } catch (error) {
    console.error("Firebase認証またはデータ取得エラー:", error);
    throw error;
  }
};

/**
 * ユーザーデータを更新する
 * @param {string} userId - ユーザーのID
 * @param {Object} updateData - 更新するデータ
 * @returns {Promise<void>}
 */
export const updateUserData = async (userId, updateData) => {
  try {
    const userRef = doc(db, 'users', userId);
    await updateDoc(userRef, updateData);
  } catch (error) {
    console.error("ユーザーデータ更新エラー:", error);
    throw error;
  }

};
