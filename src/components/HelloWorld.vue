<template>
  <div class="hello">
    <div class='container header status' :class='`${status}`'>
      <div class='row text-center'>
        <div class='col-12 text-center'>
          <h5>How is Trump doing?</h5>
          <h1>{{status}}</h1>
        </div>
      </div>
    </div>

    <div class='container header'>
      <div class='row text-center'>
        <div class='col-12 col-md-6'>
          <h2>{{currentApproval}} %</h2>
          Approve
        </div>

        <div class='col-12 col-md-6'>
          <h2>{{currentDispproval}} %</h2>
          Disapprove
        </div>
      </div>
    </div>

    <div class='container chart'>
      <div class='row text-center'>
        <div class='col-12'>
          <canvas id="myChart"></canvas>
        </div>
      </div>
    </div>

    <!-- <div class='container'>
      <div class='row text-left'>
        <div class='col-12 col-md-4' v-for='resource in resources'>
          <div class='card'>
          </div>
        </div>
      </div>
    </div> -->

    <div class='link'>
      <span>
        Data thanks to
        <a href='https://github.com/fivethirtyeight/data/tree/master/trump-approval-ratings'>fivethirtyeight.com</a>
      </span>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js';
import axios from 'axios';
import { ModelSelect } from 'vue-search-select';

const parse = require('csv-parse');

export default {
  name: 'HelloWorld',
  components: {
    ModelSelect,
  },
  metaInfo() {
    const title = 'How is Trump doing?';

    return {
      title,
      meta: [
        { vmid: 'description', name: 'description', content: title },
      ],
    };
  },
  data() {
    return {
      msg: 'Machine Learning Tutorials',
      currentApproval: 0,
      currentDispproval: 0,
    };
  },
  async mounted () {
    const response = await axios.get('https://projects.fivethirtyeight.com/trump-approval-data/approval_polllist.csv');
    const rows = response.data.split('\n');
    const stats = rows.map(row => row.split(','));

    const headers = stats[0];

    const newStats = stats.slice(1, stats.length - 1);

    const statsAverages = {};
    newStats.forEach(stat => {
      const endDate = stat[4];
      const approve = parseFloat(stat[13]);
      const disapprove = parseFloat(stat[14]);

      if (!statsAverages[endDate]) {
        statsAverages[endDate] = {
          count: 1,
          approve,
          disapprove,
        }
        return;
      }

      statsAverages[endDate].approve = (statsAverages[endDate].approve + approve) / 2;
      statsAverages[endDate].disapprove = (statsAverages[endDate].disapprove + disapprove) / 2;
    });


    const approvals = [];
    const disapprovals = [];
    const labels = Object.keys(statsAverages);
    for (let key in statsAverages) {
      const current = statsAverages[key];
      approvals.push(current.approve)
      disapprovals.push(current.disapprove)
    }

    this.currentApproval = approvals[approvals.length - 1].toFixed(2);
    this.currentDispproval = disapprovals[disapprovals.length - 1].toFixed(2);

    const ctx = document.getElementById('myChart');
    const myChart = new Chart(ctx, {
      type: 'line',
      data: {
        labels,
        datasets: [
          {
            label: 'Approvals over Time',
            data: approvals,
          },
          {
            label: 'Disapprovals over Time',
            data: disapprovals,
          }
        ]
      },
      options: {
        scales: {
            xAxes: [{
                time: {
                    unit: 'month'
                },
                ticks: {
                  autoSkip: true,
                  maxTicksLimit: 20
                },
            }]
        }
      },
    });
  },
  computed: {
    resources() {
      const result = this.$store.getters.filterItems(this.searchTerm);
      return result;
    },
    status() {
      if (this.currentApproval > this.currentDispproval) {
        return 'Approved';
      } else if (this.currentDispproval > this.currentApproval) {
        return 'Disapproved';
      }

      return 'Undecided';
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1 {
      font-size: 10vw;
  }

  .Approved h1 {
    color: #F34D30;
  }

  .Disapproved h1 {
    color: #3AA9DE;
  }

  h2 {
    font-size: 60px;
  }

  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }

  h2 {
    margin-top: 2em;
  }

  .logo {
    width: 125px;
    height: 125px;
    margin-bottom: 1em;
  }

  .status {
    margin-top: 3em;
  }

  .header {
    /* background-image: url('https://cdn.shopify.com/s/files/1/2384/1915/files/OLOGIESTITLESLIDEwithmushroom_2048x.png?v=1517064763'); */
    background-size: contain;
    background-position: center;
    background-repeat: no-repeat;
    /* background-color: #FAF4DC; */
    margin-bottom: 1em;
  }

  .search {
    border: none;
    box-shadow: 0 4px 30px 0 rgba(223,225,230,0.5);
    padding: 1em;
    font-size: 22px;
  }

  .filter-container {
    /* background: linear-gradient(to right, #E100FF, #0066ff); */
    background-color: #f9f7e3;
  }

  .filters {
    margin-bottom: 2em;
    max-width: 1140px;
    margin: 0 auto;
    padding-top: 1em;
    padding-bottom: 1em;
    margin-bottom: 1em;
  }

  .chart {
    margin-top: 10em;
  }

  .card {
    padding: 1em;
    min-height: 380px;
    background-color: #fff;
    box-shadow: 0 4px 30px 0 rgba(223,225,230,0.5);
    border-radius: 10px;
    margin-bottom: 1em;
    border: none;
  }

  .card .img {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background-position: center;
    background-size: contain;
  }

  .badge-primary {
    /* background: #FF6D4A; */
  }

  .badge {
    font-size: 14px;
    margin-bottom: 10px;
    cursor: pointer;
    display: inline-block;
    background-color: #fafafa;
    padding: 5px 10px;
    border-radius: 40px;
    color: #ccc;
    text-transform: lowercase;
  }

  .btn-primary {
    color: #999 !important;
    margin-right: 10px;
    border: 1px solid #f3f3f3;
    text-transform: uppercase;
    border-radius: 40px;
    padding: 5px 10px;
    font-weight: 600;
    font-size: 11px;
    background-color: #f9f7e3;
  }

  strong {
    /* color: #FF6D4A; */
    font-family: 'Poppins', sans-serif;
    margin-top: .5em;
    margin-bottom: .5em;
    font-size: 20px;
    font-weight: bold;
    color: #000;
  }

  .link {
    margin-top: 2em;
    margin-bottom: 2em;
  }

  .share {
    width: 100%;
    margin-top: 1em;
  }

  .share a svg {
    height: 15px;
  }

  .share a {
    color: #ccc !important;
    width: 15px;
    height: 15px;
    display: inline-block;
    position: relative;
    margin-right: .5em;
  }
</style>
