<template>
    <div>
        <div class="container">
            <label for="perPageRecord" class="form-label">Records Per Page: {{ perPage }}</label>
            <input type="range" v-model="perPage" @input="currentPage !== 1 ? currentPage = pages : false" class="form-range" min="20" :max="initPassengers.length" step="10" id="perPageRecord">
            <div class="row">
                <div class="col-md-4 col-sm-12">
                    <select class="form-select" v-model="filter">
                        <option disabled selected>Filter By</option>
                        <option
                            v-for="(filter, index) in filters"
                            :key="index"
                         :value="filter">{{ filter }}</option>
                    </select>
                </div>
                <div class="col-md-8 col-sm-12">
                <input 
                    type="text" 
                    class="form-control" 
                    placeholder="Type to search..."
                    v-model="search"
                >
                </div>
            </div>
        </div>
        <div class="container">
            <table class="table">
                <thead>
                    <tr>
                    <th scope="col">ID</th>
                    <th scope="col">Survived</th>
                    <th scope="col">PClass</th>
                    <th scope="col">Name</th>
                    <th scope="col">Sex</th>
                    <th scope="col">Age</th>
                    <th scope="col">Siblings</th>
                    <th scope="col">Parch</th>
                    <th scope="col">Ticket</th>
                    <th scope="col">Fare</th>
                    <th scope="col">Cabin</th>
                    <th scope="col">Embarked</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Get the paginated result -->
                    <tr 
                    v-for="(passenger, index) in paginateData(filteredPassengers)"
                    :key="index"
                    >
                    <th scope="row">{{ passenger.id }}</th>
                    <th>{{ passenger.Survived }}</th>
                    <td>{{ passenger.class }}</td>
                    <th>{{ passenger.Name }}</th>
                    <th>{{ passenger.Sex }}</th>
                    <td>{{ passenger.Age ? passenger.Age : 'N/A' }}</td>
                    <td>{{ passenger.Siblings }}</td>
                    <td>{{ passenger.Parch }}</td>
                    <td>{{ passenger.Ticket }}</td>
                    <td>{{ passenger.Fare }}</td>
                    <td>{{ passenger.Cabin }}</td>
                    <td>{{ passenger.Embarked }}</td>
                    </tr>
                </tbody>
            </table>
            <nav aria-label="Page navigation example" style="overflow-x:scroll;">
                <ul class="pagination">
                    <li class="page-item"><a class="page-link" href="#" @click.stop="previousPage">Previous</a></li>
                    <li 
                        v-for="(page, index) in pages"
                        :key="index"
                        :class="currentPage === page ? 'page-item active' : 'page-item'"><a class="page-link" @click="changePage(page)">{{ page }}</a></li>
                    <li class="page-item"><a class="page-link" href="#" @click.stop="nextPage">Next</a></li>
                </ul>
            </nav>
        </div>
    </div>
</template>
<script>
export default {
  name: 'DataTable',
  data() {
      return {
          initPassengers: [],
          isLoading: false,
          search: '',
          filter: 'Name',
          filters: [
              'Name', 'Age', 'Ticket', 'Embarked', 'class',
          ],
          currentPage: 1,
          perPage: 20,
      }
  },
  computed: {
      passengers() {
          // Returned promise is proxied and this converts into an array
          return JSON.parse(JSON.stringify(this.initPassengers));
      },
      filteredPassengers() {
          // make sure regardless of the case it is still searchable
          const search = this.search.toLowerCase();
          const filter = this.filter;
          // If search is empty end.
          if (! search) {
              return this.initPassengers;
          }
            // Filters the array based on search input and selected filter
          let filteredPassengers = this.passengers.filter((passenger) => {
              // make sure that integers are read as string
              let value = passenger[filter] ? String(passenger[filter]).toLowerCase() : false;
              // match check for the search input
              let checker = value ? value.includes(search) : false;
              // if match return the passenger on the array
              if (checker) {
                  return passenger;
              } 
          });
          return filteredPassengers;
      },
      pages() {
          // Get number of pages available based on result
          return Math.ceil(this.filteredPassengers.length / this.perPage);
      },
  },
  created() {
      this.fetchPassengers();
  },
  methods: {
      // Fetches the passenger data from the given test endpoint.
      async fetchPassengers() {
          // Used raw fetch function; line can be easily be done with axios
          // store promise into the initial passengers variable
          this.isLoading = true;
          this.initPassengers = await (
              await fetch('https://coding-task.strakertranslations.com/passengers')
                .then(response => response.json())
                .then(data => {
                    this.isLoading = false;
                    return data;
                })
          );
      },
      paginateData(data) {
          // Set Initial
          //starting index to slice depending on the page
          let index = 0
          // how much the array will be sliced
            let offSet = this.perPage
          // check if first page or results are not above per page; return first n number of results
            if(this.currentPage == 1 || this.pages <= 0)  {
                index = 0
                offSet = this.perPage
            } else if(this.currentPage > data.length) {
                 index = this.currentPage - 1
                 offSet = data.length
            } else {
                // multiple the current page per page minus the n number of results to get the next page results 
                 index = this.currentPage * this.perPage - this.perPage
                 offSet = index + this.perPage
            }
            // slices the array 
          return data.slice(index, offSet);
      },
      changePage(page) {
          this.currentPage = page;
      },
      previousPage() {
          const prevPage = this.currentPage - 1;
          // check if you reached the first page
          if (prevPage <= 0) {
              return false;
          }
          this.changePage(prevPage);
      },
      nextPage() {
          const nextPage = this.currentPage + 1;
          // check if you reached the last page
          if (nextPage > this.pages) {
              return false;
          }
          this.changePage(nextPage);
      },
  }
}
</script>