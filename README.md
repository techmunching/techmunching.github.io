# techmunching.github.io
TechMunching blog - Your daily tech snack

### To add image captions just write caption in between two *'s right after img
For e.g. 
```![](/img/1*cp9haQg73JXSEN-xfrJB-g.png)*This is now the beautiful image caption*```

### For mentioning video within an article

```
<div class="vidWrapper">
  <video style="max-width:100%" autoplay muted loop>
    <source src="/img/0*cfutfYn25YFEgBvM.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>
*Caption of the video*
```
### For putting video as thumbnail of the post / article instead of image, mention video with the details

Like instead of this

```
...
image: 'img/0*-ExPwJS4GXXNwbyQ.jpeg'
...
```

Do this

```
...
video: 'img/0*-ExPwJS4GXXNwbyQ.mp4'
...
```

And rest will be taken care of by the video logic in index.html
