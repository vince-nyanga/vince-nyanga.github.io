---
title: "Weekly Lessons - Episode 3"
date: 2020-08-30
---

Here is what I learned in the past couple of days:

## View Encapsulation In Angular

Whenever I wanted to show information on Google Maps I'd use the built-in `Infowindow` and that was fine for the use cases I had. This time we had a design that made it impossible to use the default `Infowindow` so I had to create a custom popup following [this example](https://developers.google.com/maps/documentation/javascript/examples/overlay-popup). This wasn't very difficult to do. However, when I opened the map the custom popup was always at the center of the map. When I panned the map the popup would snap back to the center which was not the requirement at all. What's going on :disappointed:?

After hours of trying but to no avail I bumped across this [thread](https://forum.vuejs.org/t/google-maps-api-custom-popups-example-code-breaks-in-vuejs-single-file-components/39632/2) on the `Vue js` forum where someone had the same issue in `Vue js` and they solved it by removing the scope on the styles. Then I saw that another person had the same issue on Angular and they removed the scope as well for it to work. Ok, this is a step in the right direction but as a novice in Angular I had no clue as to how that's done. The following morning (yes, I went to bed without solving it) I ran to my colleagues for help and told showed them the `Vue js` forum and immediately one of them said 'Oh, we need to use view encapsulation'. Well, what is view encapsulation?

In Angular view encapsulation defines whether the template and styles defined within a component can affect the whole application and vice versa. There are three [encapsulation strategies](https://angular.io/api/core/ViewEncapsulation):

1. **`ViewEncapsulation.Emulated`**: This is the default encapsulation strategy. In this strategy styles from the main `HTML` propagate to the components however, styles defined in a component's `@Component` decorator are scoped to that component alone.

2. **`ViewEncapsulation.None`**: This was the encapsulation strategy that solved my problem :smile:. In this strategy styles from a component propagate back to the main `HTML` and therefore visible to all components on the page.

3. **`ViewEncapsulation.ShadowDom`**: In this strategy styles from the main `HTML` do not propagate to the component. Styles defined in the component are scoped to this component only.

This was such a huge lesson for me. I know Angular pros are like 'this is obvious' but for me it wasn't as obvious and I'm happy that now I know a little more than I knew a week ago. Slowly but surely my Angular knowledge is growing.

### References

- What is ViewEncapsulation in Angular? -- a [Dev.to](https://dev.to/monicafidalgo/what-is-viewencapsulation-in-angular-470o) post
- Angular [documentation](https://angular.io/api/core/ViewEncapsulation)
