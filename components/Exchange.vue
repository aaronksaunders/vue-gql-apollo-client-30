<template>
  <div>
    <h1>{{msg}}</h1>
    <div class="card">
      <div class="card-content ">
        <span class="card-title">Card Title</span>
        <input v-model="inputMessage.content" placeholder="Content"/>
        <input v-model="inputMessage.author" placeholder="Author"/>
        <div class="card-action">
          <button class="btn" @click="addMessage" > Add </button>
        </div>
      </div>
    </div>
    <div >
      <div v-if="isLoading" >
        <vue-simple-spinner></vue-simple-spinner>
      </div>
      <ul class="collection" v-else>
        <li class="collection-item" v-for="m in messages" :key="m.id">
          <div>{{m.content}}
          {{m.created}}</div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  import gql from 'graphql-tag'
  import Spinner from 'vue-simple-spinner'

const  GET_ALL_MESSAGES = gql`
  query getAllMessages {
    messages : getAllMessages {
      id
      content
      author
      created
    }
  }
`;

const ADD_MESSAGE = gql`
  mutation add($msgInput: MessageInput!) {
    newMessage : createMessage(input: $msgInput) {
      id
      content
      author
      created
    }
  }
`;

	export default {
    created() {
      //debugger
    },
    components: {
      // Component used to create the progress UI component
      'vue-simple-spinner':Spinner
    },
    computed: {
    },
    // specific varoables that we are using in this simple
    // application
    data () {
      return {
        msg: 'Vue.js App - GraphQL With Apollo Client 3.0',
        messages : [],
        isLoading : true,
        inputMessage : {
          content :'',
          author : ''
        }
      }
    },
    // 
    methods :{
      addMessage() {
        // We clear it early to give the UI a snappy feel
        this.newTag = ''
        // Call to the graphql mutation
        this.$apollo.mutate({
          // Query
          mutation: ADD_MESSAGE,
          // Parameters
            variables: {
              msgInput: { ...this.inputMessage, created: new Date() }
            },
          // Update the cache with the result
          // The query will be updated with the optimistic response
          // and then with the real result of the mutation
          update: (store, { data: { newMessage } }) => {
            debugger
            // Read the data from our cache for this query.
            const data = store.readQuery({ query: GET_ALL_MESSAGES })
            // Add our tag from the mutation to the end
            data.messages.push(newMessage)
            // Write our data back to the cache.
            store.writeQuery({ query: GET_ALL_MESSAGES, data })
          },
        }).then((data) => {
          // Result
          console.log(data)

          this.inputMessage = {
            content :'',
            author : ''
          }
        }).catch((error) => {
          // Error
          console.error(error)
        })
      },
    },
    // This is the code to support the apollo client
    // queryingt the data
    apollo : {
      // variable to hold results of query..
      messages :   {
        // see the actual query below...
        query: GET_ALL_MESSAGES,
        // this is where we track the loading state which 
        // is used to determine if we should show indicator
        // of not
        watchLoading(isLoading, countModifier) {
          // isLoading is a boolean
          // countModifier is either 1 or -1
          console.log("isLoading",isLoading)
          this.isLoading = isLoading
        }
      },
    },
  }


</script>

<style>
.enter { transform: translateX(100%) }
.enter-to { transform: translateX(0) }
.slide-enter-active { position: absolute }

.leave { transform: translateX(0) }
.leave-to { transform: translateX(-100%) }

.slide-enter-active,
.slide-leave-active { transition: all 750ms ease-in-out }

  .heading {
    font-size:  40px;
    font-weight: bolder;
    //color: white;
    letter-spacing: .2em;
    text-align: center;
    padding: 20px
  }

</style>

