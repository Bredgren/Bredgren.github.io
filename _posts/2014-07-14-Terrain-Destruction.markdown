---
layout: post
title:  "Terrain Destruction"
date:   2014-07-14 22:00:00
categories: update, games
---

I just spent the past weekend working on a prototype for terrain destruction. It's something I would very much like to have in my game Cosmic Combat. I attempted to implement terrain destruction by clipping polygons and then converting them to triangles for a physics engine to use. I almost got it working but the library I used for converting to triangles did not do so in a desirable fashion. Also, getting three third party libraries to work together was a bit messy. And so I took another long break from that project to work on other stuff.

Then, last week, by random chance I started thinking about terrain destruction from a new perspective. I was reminded of the python library curses, which basically lets you display things in a terminal however you want. I don't remember why but for some reason I decided I wanted to make destructible terrain and use curses to display it, just for fun. This got me thinking of terrain as made up of cells laid out in on a grid, where I would have a collection of which cells made of the terrain and I would just have to figure out how to connect them. It's hard to remember the train of thought exactly, but eventually I had come up with an algorithm that I wanted to try out. And curses was no longer going to cut it so I switched to my preferred method of prototyping: pygame.

The algorithm is relatively simple, it's less than 300 lines and can be seen in its entirety below, minus the good comments. (I can be lazy about that when working quickly on prototypes, sorry.) Actually if I remove everything except the algorithm itself then it's less than 200 lines. And it could also be made even simpler by removing support for nested loops, which I don't anticipate my game needing.

At a high level this is what it's doing (if we're not allowing nested loops):

It starts with a collection of xy-coordinates representing the cells that are part of the terrain. There are no assumed units, each cell just has a length and width of 1 and its position is its top left corner so that its vertices aren't at half units.

It then generates a set of all the vertices that make up the terrain, the 4 corners of each cell. And now the cells can be forgotten about. Perhaps this can be optimized by throwing away any notion of cells and just using vertices but the collection of vertices generated from the collection of cells has convenient properties.

The first thing to do is remove points that are obviously going to be inside a loop, like the center of a 2x2 block of cells.

Now we select a starting point. I've chosen to select the point that is furthest left then furthest up. From this point we search each location that is distance 1 away, including diagonals, in a clockwise direction starting with straight up because we know there is no point there. (From the start the next point is guaranteed to be at the upper right diagonal or straight right.) Then we move on to the next point and repeat the process, except we choose a different direction to start the clockwise search from. We start pointing in the direction of the previous point but shifted one "search direction" clockwise so we don't go back to where we came from. So if the second point was directly right of the first, then the first direction that the second point searches in, for the third point, is the point directly above the first point. (Now that I think about it may be possible to skip that first search at every point, because if there was point there then the previous point would have hit it instead...maybe. I would need to think about it some more.) This continues until we return to the starting point, which we are guaranteed to do (unless you start nesting loops).

After a loop is found we remove all the points that are part of that loop, and all the points inside that loop, from the collection of vertices and begin again until there are no more.

My algorithm then processes all the loops to remove unnecessary vertices.

I make this with Box2D in mind because of its chain shape. My algorithm's output should be easily converted directly into chain shapes.

Here's a quick demo video followed by the source code.

<iframe width="560" height="315" src="//www.youtube.com/embed/kQSBdmrvK3M?list=PLqEImp3lhrWxz77dF1AqCycgZ6QuWVPCa" frameborder="0" allowfullscreen></iframe>

{% highlight python %}

import random, math, time

class CellTerrain:
    """The cell's position is its top left corner"""
    pattern = []

    def __init__( self ):
        self._cells = set()
        self._loops = set()
        self._edges = set()
        self._allow_holes = False
        self._extra_verts = set()
        self._inner_verts = set()
        self._error_verts = set()
        if not self.pattern:
            CellTerrain._construct_pattern()

    def add_cells( self, cells ):
        """cells - collection of (x, y)"""
        self._check_and_update( lambda: self._cells.update( cells ) )

    def allow_holes( self ):
        return self._allow_holes

    def allow_holes_is( self, value ):
        old = self._allow_holes
        self._allow_holes = value
        if old != value:
            self._update()

    def cells( self ):
        return self._cells

    def cells_is( self, cells ):
        self._cells = set( cells )

    def clear_cells( self ):
        self._cells.clear()
        self._clear()

    def error_verts( self ):
        return self._error_verts

    def explode( self, pos, size ):
        """pos - cell position to center the explosion at
           size - radius in cell units"""
        to_remove = set()
        for cell in self._cells:
            dist_x = cell[ 0 ] - pos[ 0 ]
            dist_y = cell[ 1 ] - pos[ 1 ]
            dist2 = dist_x ** 2 + dist_y ** 2
            if dist2 < ( size ** 2 + random.randint( -5, 5 ) ):
                to_remove.add( cell )

        self.remove_cells( to_remove )

    def extra_verts( self ):
        return self._extra_verts

    def inner_verts( self ):
        return self._inner_verts

    def loops( self ):
        return self._loops

    def remove_cells( self, cells ):
        """cells - collection of (x, y)"""
        self._check_and_update(
            lambda: self._cells.difference_update( cells ) )

    def _check_and_update( self, fn ):
        old = len( self._cells )
        fn()
        if old != len( self._cells ):
            self._update()

    def _clear( self ):
        self._loops.clear()
        self._extra_verts.clear()
        self._inner_verts.clear()
        self._error_verts.clear()

    def _find_loop( self, start, verts, reverse ):
        loop = [ start ]
        pos = start
        if reverse:
            prev = ( pos[ 0 ] - 1, pos[ 1 ] )
        else:
           prev = ( pos[ 0 ], pos[ 1 ] - 1 )
        next_vert = self._find_next( pos, prev, verts, reverse )
        while next_vert and next_vert != start:
            prev = pos
            pos = next_vert
            loop.append( pos )
            next_vert = self._find_next( pos, prev, verts, reverse )
            if not next_vert: break

        for vert in loop:
            if not next_vert:
                print "error:", vert
                self._error_verts.add( vert )
            if vert in verts:
                verts.remove( vert )

        return loop

    def _find_loops( self, verts, reverse=False ):
        def _rev_cmp(a, b):
            if a[0] > b[0]: return -1
            elif a[0] < b[0]: return 1
            elif a[1] < b[1]: return -1
            elif a[1] > b[1]: return 1
            else: return 0
        loops = []
        if reverse:
            sorted_verts = sorted( verts, cmp=_rev_cmp )
        else:
            sorted_verts = sorted( verts )
        start = sorted_verts[ 0 ]
        loop = self._find_loop( start, verts, reverse )
        loops.append( loop )

        inner_verts = self._verts_in_loop( loop, verts )
        verts -= inner_verts
        if self._allow_holes:
            # Look for loops within the current loop
            if inner_verts: # base case here...
                loops += self._find_loops( inner_verts, not reverse )
        else:
            # Not allowing holes so remove inner verts from consideration
            self._inner_verts.update( inner_verts )

        # Look for loops outside the current loop (at the same 'level')
        if verts: # ...and here because otherwise I would be appending empty lists
            loops += self._find_loops( verts, reverse )

        return loops

    def _find_next( self, current, previous, verts, reverse ):
        prev_dir = ( ( previous[ 0 ] - current[ 0 ] ),
                     ( previous[ 1 ] - current[ 1 ] ) )
        prev_angle = math.atan2( prev_dir[ 1 ], prev_dir[ 0 ] )
        pattern = self.pattern
        if reverse:
            pattern = list( reversed( pattern ) )
            pattern = pattern[ 1: ] + pattern[ :1 ]
        start = 0
        for x, y in pattern:
            if ( ( reverse and ( math.atan2( y, x ) < prev_angle ) ) or
                ( not reverse and ( math.atan2( y, x ) > prev_angle ) ) ):
                break
            start += 1

        start = start % len( pattern )
        for i in ( range( start, len( pattern ) ) + range( 0, start ) ):
            new_dir = pattern[ i ]
            new_vert = ( current[ 0 ] + new_dir[ 0 ],
                         current[ 1 ] + new_dir[ 1 ] )
            if new_vert in verts:
                return new_vert

        print "Couldn't find next"
        return False

    def _verts_in_loop( self, loop, verts ):
        inner = set()
        for vert in verts:
            if self._vert_in_loop( vert, loop ):
                inner.add( vert )

        return inner

    def _remove_extra( self, verts ):
        """Looks at all verticies and removes those that are guarenteed to
           be considered inside a loop."""
        def _enclosed( vert ):
            diagonals = [
                ( vert[ 0 ] - 1, vert[ 1 ] - 1 ) in verts,
                ( vert[ 0 ] - 1, vert[ 1 ] + 1 ) in verts,
                ( vert[ 0 ] + 1, vert[ 1 ] - 1 ) in verts,
                ( vert[ 0 ] + 1, vert[ 1 ] + 1 ) in verts ]

            edges = ( ( vert[ 0 ], vert[ 1 ] - 1 ) in verts and
                      ( vert[ 0 ], vert[ 1 ] + 1 ) in verts and
                      ( vert[ 0 ] - 1, vert[ 1 ] ) in verts and
                      ( vert[ 0 ] + 1, vert[ 1 ] ) in verts )

            return edges and diagonals.count( True ) >= 2

        to_remove = set()
        for vert in verts:
            if _enclosed( vert ):
                to_remove.add( vert )
                self._extra_verts.add( vert )

        verts -= to_remove

    def _simplify_loops( self, loops ):
        def _simplify_loop( loop ):
            if len( loop ) <= 1: return loop
            new_loop = [ loop[ 0 ] ]
            prev = loop[ 1 ]
            x_dir = loop[ 1 ][ 0 ] - loop[ 0 ][ 0 ]
            y_dir = loop[ 1 ][ 1 ] - loop[ 0 ][ 1 ]
            for vert in loop[ 2: ] + [ loop[ 0 ] ]:
                new_x_dir = vert[ 0 ] - prev[ 0 ]
                new_y_dir = vert[ 1 ] - prev[ 1 ]

                if new_x_dir != x_dir or new_y_dir != y_dir:
                    new_loop.append( prev )

                x_dir = new_x_dir
                y_dir = new_y_dir
                prev = vert

            return new_loop

        new_loops = []
        for loop in loops:
            new_loops.append( _simplify_loop( loop ) )

        return new_loops

    def _update( self ):
        start_t = time.time()

        verts = set()
        self._clear()
        for cell in self._cells:
            verts.add( ( cell[ 0 ], cell[ 1 ] ) )
            verts.add( ( cell[ 0 ] + 1, cell[ 1 ] ) )
            verts.add( ( cell[ 0 ], cell[ 1 ] + 1 ) )
            verts.add( ( cell[ 0 ] + 1, cell[ 1 ] + 1 ) )

        if not verts: return

        self._remove_extra( verts )
        loops = self._find_loops( verts )
        loops = self._simplify_loops( loops )
        loops = [ tuple( l ) for l in loops ]
        self._loops.update( loops )

        end_t = time.time()
        total_t = end_t - start_t

    def _vert_in_loop( self, vert, loop ):
        """This was translated from this website:
    http://www.ecse.rpi.edu/Homepages/wrf/Research/Short_Notes/pnpoly.html"""
        nvert = len( loop )
        vertx, verty = zip( *loop )
        testx = vert[ 0 ]
        testy = vert[ 1 ]
        i = 0
        j = nvert - 1
        c = False
        while i < nvert:
            if ( ( ( verty[ i ] > testy ) != ( verty[ j ] > testy ) ) and
                 ( testx < ( vertx[ j ] - vertx[ i ] ) *
                   ( testy - verty[i] ) / ( verty[j] - verty[ i ] ) +
                   vertx[i] ) ):
                c = not c
            j = i
            i += 1

        return c

    @staticmethod
    def _construct_pattern():
        def get_pattern( A, B ):
            def minus( l ):
                return [ -i for i in l ]
            px = A + B + minus( A ) + minus( B )
            py = minus( B ) + A + B + minus( A )
            return zip( px, py )

        A1 = [ 0, 1 ]
        B1 = [ 1, 1 ]

        pattern1 = get_pattern( A1, B1 )
        CellTerrain.pattern = pattern1

{% endhighlight %}
