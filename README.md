## Map Index (MIT License)
Fast map clusterization build on top of [Region QuadTree](http://en.wikipedia.org/wiki/Quadtree).

## Requirements 
* Minimum iOS 6.0+ due to changes in MKMapView callback mechanics. 
* MapKit.framework

## Usage
Replace MKMapView with MIMapView instance & enjoy ;) 

To provide own annotation views for cluster use:
```objective-c
- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id <MKAnnotation>)annotation
{
  if ([annotation isKindOfClass:[MIAnnotation class]])
  {
    MKAnnotationView *clusterView = ...;

    ...

    return clusterView;
  }
    
  return ...;
}

```


## Custom animation
You're able to provide <b>your own animation<b> for zoom-in, move, zoom-out. 
For these puproses MIMapView declare <br>@property (nonatomic, strong) MITransactionFactory *transactionFactory;

How to use: <br>1. Subclass <br>2. Change default transactions factory to your one

```objective-c
- (MITransaction *)transactionWithTarget:(NSArray *)target source:(NSArray *)source order:(NSComparisonResult)order
{
	switch (order)
	{
		case NSOrderedSame:
			return transaction for region change without zoom-in/zoom-out
			break;

		case NSOrderedAscending:
  		return transaction for zoom-in
			break;

		case NSOrderedDescending:
			return transaction for zoom-out
			break;
	}
}
```

###MITransaction
This class is responsible for add/remove of target annotations.
Each instance provide target annotations (which you have to add) and source annotations (which you have to remove).

Purpose of this class is to handle annotations change animated without any pain.
###MITransaction Subclass
In case of custom transaction you have to implement next methods:

```objective-c
- (void)perform
{
  // Transaction invoked. 
  // For example if you're implementing zoom-in transaction - you need to add target annotations
  if (self.target.count > 0)
	{
		[self addAnnotations:self.target];
	}
	else if (self.source.count > 0)
	{
    // Reason, why we're removing annotaitons here, not in - mapView:didAddAnnotationViews:  
    // Is that next method get called only if you've added some annotations. 
    // Therefore if you're going to remove source annotations later but have no new annotations - ooops:)

		[self removeAnnotations:self.source];
	}
}

- (void)mapView:(MIMapView *)mapView didAddAnnotationViews:(NSArray *)views
{
  // This method get called only if you've added any annotations.
  // Therefore take in account any code, that should be executed here. 
} 
```

<b>Important notes:<b>
Animation usually takes longer than 0 seconds. 
To prevent map's data change during animation you can lock/unlock transaction in appropriate places.
It guarantees, that target annotations still on the map (but your code deleted some of them)
Example:

```objective-c
- (void)mapView:(MIMapView *)mapView didAddAnnotationViews:(NSArray *)views
{
  [self lock];

  [views makeObjectsPerformSelector:@selector(setAlpha:) withObject:nil];

	[UIView animateWithDuration:_MIRegularTransactionDuration animations:^
	{
		for (MKAnnotationView *view in views)
		{
			[view setAlpha:1.f];
		}

	} completion:^(BOOL finished)
	{
		[self unlock];
	}];
} 
```


###Special thanks to SuperPin: 
I've used Airports.plist & Airport.m from their DEMO. They also inspired me to rewrite my Obj-C implementation of QuadTree (which was incredibly slow). All Profiling stuff has been done in comparsion with SuperPin, so big thanks to them.
