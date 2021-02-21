<template>
  <div class="container">
    <div class="row justify-content-md-center pt-4">
      <div class="col-12">
        <div class="card">
          <div class="card-header text-center bg-white">
            <p class="h1">Error List</p>
          </div>
          <div class="row mb-3 p-2">
            <div v-if="cachedAction.length > 0" class="col-12">
              <div class="error__btn-undo"></div>
              <button
                type="button"
                class="btn bg-danger btn-lg text-white mb-2"
                @click="onClickUndo()"
              >
                Undo
              </button>
            </div>
            <div class="col-4">
              <div class="card">
                <div class="card-header mb-2 text-center bg-warning">
                  <p class="text-white">Unresolved</p>
                </div>
                <div class="row">
                  <div class="col-12">
                    <div class="error__container unresolved__bg" v-for="error in unresolved" :key="error.index">

                      <div class="error__text">{{ error.text }}</div>
                      <div class="error__code bg-warning text-white">{{ error.code }}</div>
                      <div class="error__btn">
                        <button
                          type="button"
                          class="btn btn-success"
                          @click="onClickResolve(error)"
                        >Resolve
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div class="col-4">
              <div class="card">
                <div class="card-header mb-2 text-center bg-success">
                  <p class="text-white">Resolved</p>
                </div>
                <div class="row">
                  <div class="col-12">
                    <div class="error__container resolved__bg" v-for="error in resolved" :key="error.index">

                      <div class="error__text">{{ error.text }}</div>
                      <div class="error__code bg-success text-white">{{ error.code }}</div>
                      <div class="error__btn">
                        <button
                          type="button"
                          class="btn bg-warning text-white"
                          @click="onClickUnresolve(error)"
                        >Unresolve
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div class="col-4">
              <div class="card">
                <div class="card-header mb-2 text-center bg-info">
                  <p class="text-white">Backlog</p>
                </div>
                <div class="row">
                  <div class="col-12">
                    <div class="error__container backlog__bg" v-for="error in backlog" :key="error.index">

                      <div class="error__text">{{ error.text }}</div>
                      <div class="error__code bg-info text-white">{{ error.code }}</div>
                      <div class="error__btn">
                        <button
                          type="button"
                          class="btn bg-warning text-white"
                          @click="onClickUnresolveFromBacklog(error)"
                        >Unresolve
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  async asyncData({$axios}) {
    try {
      const {resolved, unresolved, backlog} = await $axios.$get(
        "http://localhost:8000/get_lists"
      );
      return {
        resolved,
        unresolved,
        backlog
      };
    } catch (error) {
      console.log(
        `Couldn't get error lists:\n${error}\nDid you start the API?`
      );
      console.log(
        "HINT: You can comment out the full `asyncData` method and work with mocked data for UI/UX development, if you want to."
      );
    }
  },
  data() {
    return {
      resolved: [],
      unresolved: [],
      backlog: [],
      cachedAction: []
    };
  },
  methods: {
    onClickResolve(selectedError) {

      this.cachedAction.unshift(
        {
          propName: 'unresolved',
          data: [...this.unresolved]
        },
        {
          propName: 'resolved',
          data: [...this.resolved]
        }
      );

      const error = this.unresolved.find(err => err.index === selectedError.index);

      this.unresolved = this.unresolved.filter(err => err.index !== error.index)

      this.resolved = [...this.resolved, error]

      //TODO Make a call for RESOLVE
    },
    onClickUnresolve(selectedError) {

      this.cachedAction.unshift(
        {
          propName: 'unresolved',
          data: [...this.unresolved]
        },
        {
          propName: 'resolved',
          data: [...this.resolved]
        }
      );

      const error = this.resolved.find(err => err.index === selectedError.index);

      this.resolved = this.resolved.filter(err => err.index !== error.index)

      this.unresolved = [...this.unresolved, error]

      //TODO Make a call for UNRESOLVE
    },
    onClickUnresolveFromBacklog(selectedError) {

      this.cachedAction.unshift(
        {
          propName: 'backlog',
          data: [...this.backlog]
        },
        {
          propName: 'unresolved',
          data: [...this.unresolved]
        }
      );

      const error = this.backlog.find(err => err.index === selectedError.index);

      this.backlog = this.backlog.filter(err => err.index !== error.index)

      this.unresolved = [...this.unresolved, error]
    },
    onClickUndo() {

      const [first, second] = this.cachedAction;
      this.[first.propName] = first.data;
      this.[second.propName] = second.data;

      this.cachedAction = this.cachedAction.slice(2);

      //TODO Make a call for UNDO
    }
  }
};
</script>
<style>
.unresolved__bg {
  background-color: #fff3cd;
}

.resolved__bg {
  background-color: #d1e7dd;
}

.backlog__bg {
  background-color: #cff4fc;
}

.error__container {
  display: flex;
  padding: 5px 10px;
  margin: 5px 10px;
  border-radius: 5px;
}

.error__code {
  border-radius: 5px;
  max-height: 25px;
  font-weight: 700;
  font-size: 16px;
  padding: 0 7px;
  align-self: center;
}

.error__text {
  font-size: 16px;
  margin: 5px 10px;
}

.error__btn {
  align-self: center;
  margin: 5px;
}

.error__btn-undo {
  display: flex;
  flex-direction: column;
  padding: 0;
  margin: 0;
}
@media (max-width: 1190px) {
  .error__container {
    flex-direction: column;
  }
  .error__text{
    margin: 1px;
    font-size: 11px;
  }
  .error__code{
    font-size: 11px;
  }
}
</style>
