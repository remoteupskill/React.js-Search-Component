# complete frontend search flow

First we must think about how to organise the different parts parts of the search logic into actions, services and components:

- Search.js
- SearchService.js
- SearchActions.js
- SearchJobs.js
- SearchResults.js
- APIService.js

Let's in detail what is the prupose of each one with code examples!

# Search.js

- this is the component that brings everything together

# APIService.js

- we use the file to house all the different types of api functions we will be amking in our frontend app.

- This is a resuable service that can be used anywhere on the frontend to make api calls for example making a PUT or GET request to a Mongo database on the backend

- we can use the service to set all the headers and handle and errors returned from the backend using axios.interceptors.response.use

```
import axios from 'axios'

class BaseService {
  constructor() {
    axios.interceptors.response.use(response => {
      return response
    }, error => {
      if (error.response) {
        if (error.response.status === 401) {
          try {
            localStorage.clear()
            const { pathname, search } = document.location
            const referrer = pathname + encodeURIComponent(search)
            window.location.href = `/login-page`
            return true
          } catch (e) {
            return false
          }
        } else if (error.response.status === 403) {
          try {
            window.location.href = '/unathorized'
            return true
          } catch (e) {
            return false
          }
        }
      }
      return Promise.reject(error)
    })
}

get = (url, params, headers) => {
    return axios({
      method: 'get',
      headers: this.headers,
      url,
      params: params || {},
      responseType: 'json',
    })
  }
}

export default APIService
```

# SearchActions.js

- we use this file to put all redux actions related to search

- when the search is successful we dispatch a search success action

- the search success action can then trigger a function to save the search results to local storage to be used later and to reduce the number of api calls made to our backend

- at the start of our search we can execute an action called 'SEARCH_JOBS' to show a loading spinner in the UI so the user knows we are trying to fecth the data from our backend

- then wehn the response comes back we either display an error or do something with the search results

```
export const afterSearch = (type, searchSuccess) => ({
  type: type,
  searchSuccess,
})

export function searchJobs(params, page, limit) {
  return (dispatch) => {
    dispatch('SEARCH_JOBS')
    return SearchService.searchJobs(params, page, limit)
      .then(res => {
        const response = res.data.data
        dispatch(afterSearch(`SEARCH_SUCCESS`, response))
      })
      .catch(error => {
        dispatch(afterSearch(`SEARCH_FAILED`, error))
      })
  }
}

```

# SearchService.js

- we use this file to make api calls to our backend

```
import APIService from './APIService'

class SearchService extends APIService {

  searchJobs = ({ searchString }, page = 1, limit = 20) => {

    const url = '/search/jobs'

    const params = { page, limit, searchString }

    return this.get(url, params)
  }
}

export default new SearchService()
```
