---
title: "Gists/Snippets"
layout: post
---

{% highlight c++ %}
// Normalize an input value to 0,1 range.
f32 Normalize(f32 Input, f32 Minimum, f32 Maximum)
{
    return (Input - Minimum) / (Maximum - Minimum);
}

// Map one range onto another. Ex: Map Value Between 0-1800 to -20, 20.
f32 Remap(f32 Input, f32 InputStart, f32 InputEnd, f32 OutputStart, f32 OutputEnd)
{
    return (Input - InputStart) / (InputEnd - InputStart) * (OutputEnd - OutputStart) + OutputStart;
}
{% endhighlight %}

## GLSL
{% highlight glsl %}
// Grayscale
FragmentColor = texture(SampledTexture, UV);
float Average = (0.2126 * FragmentColor.r) + (0.7152 * FragmentColor.g) + (0.0722 * FragmentColor.b);
FragmentColor = vec4(Average, Average, Average, 1.0);

// Gamma Correction
const float Gamma = 2.2;
FragmentColor = pow(FragmentColor, vec3(1.0 / Gamma));
{% endhighlight %}