<template>
    <nav class="navbar fixed-top navbar-toggleable-sm navbar-light bg-faded">
      <button class="navbar-toggler navbar-toggler-right" type="button" data-toggle="collapse" data-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>
      <a class="navbar-brand" href="#">MIT HOOD</a>
      <div class="collapse navbar-collapse" id="navbarNavDropdown">
        <ul class="navbar-nav ml-auto">
          <li class="nav-item dropdown" v-show="showToc">
            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdownMenuLink" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
              GÅ TIL
            </a>
            <div class="dropdown-menu dropdown-menu-right" aria-labelledby="navbarDropdownMenuLink">
                <a class="dropdown-item" href="#" v-scroll-to="'#mapid'"><i class="fa fa-map-marker" aria-hidden="true"></i> Kort</a>
                <a class="dropdown-item" href="#" v-for="item in data" v-show="item.tablename !== 'TotalNiveau' && item.tablename !=='TotalKom'" v-scroll-to="'#' + item.tablename">{{item.title}}</a>
            </div>
          </li>
        </ul>
      </div>
    </nav>
</template>

<script>
export default {
  props: ['data', 'showToc'],
  name: 'navbar'
}
</script>

<style scoped>
</style>
