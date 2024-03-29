<script setup lang="ts">
import { useField, useForm } from 'vee-validate'
import { useUserStore } from '~/store/user'
import { SignInResponse } from '~/types/api'

// 環境変数（.env参照）からAPIのベースURLを取得
const $config = useRuntimeConfig()
const apiBaseUrl = $config.public.apiBaseUrl
// ユーザーストアを取得
const userStore = useUserStore()
// 転送処理を行うためのフック
const $router = useRouter()

// フォームの設定
const { handleSubmit, isSubmitting } = useForm({
  // バリデーションルール
  validationSchema: {
    email(value: string) {
      // 必須
      if (!value) {
        return 'メールアドレスを入力してください'
      }
      // メールアドレスの形式
      const emailPattern =
        /^[a-zA-Z0-9_+-]+(\.[a-zA-Z0-9_+-]+)*@([a-zA-Z0-9][a-zA-Z0-9-]*[a-zA-Z0-9]*\.)+[a-zA-Z]{2,}$/
      if (!emailPattern.test(value)) {
        return 'メールアドレスの形式が正しくありません'
      }
      return true
    },
    password(value: string) {
      if (!value) {
        return 'パスワードを入力してください'
      }
      return true
    },
  },
})
// フィールドの値とエラーメッセージを取得
const { value: email, errorMessage: emailErrorMessage } = useField('email')
const { value: password, errorMessage: passwordErrorMessage } =
  useField('password')

/**
 * サインイン処理
 */
const onSubmit = handleSubmit(async () => {
  try {
    // ログインAPIを呼び出す -> /backend/src/routes/auth/auth.router.tsを参照
    const { data } = await useFetch<SignInResponse>(
      `${apiBaseUrl}/auth/signin`,
      {
        method: 'POST',
        body: {
          email: email.value,
          password: password.value,
        },
      }
    )

    // レスポンスのデータを取得（ref値）
    const response = data.value
    // トークンの有無でログインできたか判断
    const hasToken = response && !!response.token
    if (hasToken) {
      // 成功の場合はトークンを保存
      userStore.token = response.token
      userStore.email = response.email
      // トップページに遷移
      $router.push('/')
    } else {
      // 失敗の場合はフィールドをクリア
      email.value = ''
      password.value = ''
    }
  } catch (error) {
    console.log(error)
  }
})

// ページのタイトルなどを設定
useHead({
  title: 'サインインページ',
})
</script>

<template>
  <div>
    <h2>サインイン</h2>
    <form @submit="onSubmit">
      <input v-model="email" name="email" type="email" />
      <p class="error">{{ emailErrorMessage }}</p>
      <input
        v-model="password"
        name="password"
        type="password"
        autocomplete="on"
      />
      <p class="error">{{ passwordErrorMessage }}</p>
      <button type="submit" :disabled="isSubmitting">送信</button>
    </form>
  </div>
</template>

<style lang="scss" scoped>
h2 {
  font-size: 1.8rem;
  @media #{$pc} {
    font-size: 2.4rem;
  }
}
.error {
  /* ~/assets/style/_variables.scssで定義した変数を使うことができます */
  color: $colorRed;
}
</style>
