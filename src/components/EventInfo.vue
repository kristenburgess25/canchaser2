<template>
  <div class="hello">
      THIS IS THE EVENT INFO PAGE
  </div>
</template>

<script>
import axios from 'axios'
import cheerio from 'cheerio'

export default {
    name: 'eventinfo',
    data () {
        return {
            events: [],
        }
    },
     created: function () {
        this.getBaseInfo()
    },
    methods: {
        getDetails: function(event) {
        // console.log('hit get event details', event)
        const detailLink = event.link
        axios(detailLink)
        .then(response => {
            const html = response.data;
            const $ = cheerio.load(html)
            const detailsTable = $('tbody > tr')
            const details = {
                description: '',
                contact: '',
            }
  
            detailsTable.each(function() {

                const rightColText = $(this).find('.text-right').text()
                const siblings = $(this).find('.text-right').siblings().text()
  
  
                if(rightColText === 'Description') {
                    details.description = siblings
                }
  
                if(rightColText === 'Contact') {
                    details.contact = siblings
                }
            })
            const mergedData = {...event, ...details}
            this.events.push(mergedData)
        })
        .catch();

    },
    getBaseInfo: function () {
        const ref = this
        axios('https://barrelhorseworld.com/events.asp')
      .then(response => {
        const html = response.data;
        const $ = cheerio.load(html)
        const eventsTable = $('tbody > tr');
        const parentTable = $('tbody')
        const trimmedTable = eventsTable.slice(1);
        console.log('events table length before', parentTable.children())   
        // parentTable.removeChild(parentTable.firstChild)
        // delete eventsTable[0]
        // console.log('parent table length after ', eventsTable.length()) 
        
        trimmedTable.each(function () {
        
          const rowData = $(this).find('td');
          const eventTitle = $(rowData).find('a').text()
          const detailLink = $(rowData).find('a').attr('href')
          const eventDate = $(rowData).first().text()
          const eventLocation = $(rowData).last().text()

          const detailUrl = `https://barrelhorseworld.com/${detailLink}`

          const event = {
            title: eventTitle,
            date: eventDate,
            location: eventLocation,
            link: detailUrl
          }
          ref.getDetails(event);
        });
      })
      .catch();
    }
  },
  props: {
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
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
}
</style>
