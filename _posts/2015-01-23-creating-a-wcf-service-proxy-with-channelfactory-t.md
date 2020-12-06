---
layout: post
title: "Creating a WCF service proxy with ChannelFactory <T>"
subtitle: "Creating a WCF service proxy with ChannelFactory <T>"
date: 2015-01-23 10:44
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, Visual Studio, WCF]
tags: [.Net, C#, C#.Net, WCF, WCF Generic Proxy]
header-img: "img/post-bg-01.jpg"
---
A proxy, in its most general form, is a class functioning as an interface to something else. The proxy could interface to anything: a network connection, a large object in memory, a file, or some other resource that is expensive or impossible to duplicate.(From [wiki](http://en.wikipedia.org/wiki/Proxy_pattern)). In case of WCF, proxy class helps client application to use WCF service,without knowing the address or implementation details. Here is the code for generic WCF proxy, using ChannelFactory class. ChannelFactory enables you to dynamically creating a proxy object based on the Service Contract. It will help you to avoid update service references or proxies generated using svcutil utility.

{% highlight CSharp %}
internal sealed class GenericProxy<TContract> : IDisposable where TContract : class
{
    private ChannelFactory<TContract> _channelFactory;
    private TContract _channel;

    public GenericProxy()
    {
        _channelFactory = new ChannelFactory<TContract>();
    }

    public GenericProxy(Binding binding, EndpointAddress remoteAddress)
    {
        _channelFactory = new ChannelFactory<TContract>(binding, remoteAddress);
    }

    public void Execute(Action<TContract> action)
    {
        action.Invoke(Channel);
    }

    public TResult Execute<TResult>(Func<TContract, TResult> function)
    {
        return function.Invoke(Channel);
    }

    private TContract Channel
    {
        get
        {
            if (_channel == null)
            {
                _channel = _channelFactory.CreateChannel();
            }

            return _channel;
        }
    }

    public void Dispose()
    {
        try
        {
            if (_channel != null)
            {
                var currentChannel = _channel as IClientChannel;
                if (currentChannel.State == CommunicationState.Faulted)
                {
                    currentChannel.Abort();
                }
                else
                {
                    currentChannel.Close();
                }
            }
        }
        finally
        {
            _channel = null;
            GC.SuppressFinalize(this);
        }
    }
}
{% endhighlight %}

The IDisposable implementation will help developers to close / abort channel with "using" statement.

And you can use this like the following.

{% highlight CSharp %}
using (var proxy = new GenericProxy<ICalculator>())
{
    var result = proxy.Execute<int>(x => x.Add(1, 2));
    return result;
}
{% endhighlight %}

The code is invoking Add method of Calculator service. The EndPoint and Binding information will fetched from the app.config file.

Happy Programming :)
