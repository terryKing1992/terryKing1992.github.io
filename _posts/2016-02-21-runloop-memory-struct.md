---
layout: post
title: Runloop内存结构
date: 2016-04-20 21:46:52
tags: [Runloop, Thread Programming]
---
	```json
	<CFRunLoop 0x7fb9f1600540 [0x10b29b7b0]>
	{
		wakeup port = 0x1503, stopped = false, ignoreWakeUps = false,
		current mode = UIInitializationRunLoopMode,
		common modes = <CFBasicHash 0x7fb9f1707ef0 [0x10b29b7b0]>
		{
			type = mutable set, count = 2,

			entries =>

				0 : <CFString 0x10c196270 [0x10b29b7b0]>{contents = "UITrackingRunLoopMode"}
				2 : <CFString 0x10b2bbb60 [0x10b29b7b0]>{contents = "kCFRunLoopDefaultMode"}
		},

		common mode items = <CFBasicHash 0x7fb9f1708bf0 [0x10b29b7b0]>
		{
			type = mutable set, count = 16,

			entries =>value_name: value,

				0 : <CFRunLoopSource 0x7fb9f16006b0 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
					{
						version = 0, info = 0x0, callout = PurpleEventSignalCallback (0x10e82f779)
					}
				}

				1 : <CFRunLoopSource 0x7fb9f14008b0 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f170d9a0 [0x10b29b7b0]>
					{
						valid = Yes, port = 1e03, source = 0x7fb9f14008b0, callout = __IOHIDEventSystemClientQueueCallback (0x10d4b987e), context = <CFMachPort context 0x7fb9f170d6d0>
					}
				}

				2 : <CFRunLoopObserver 0x7fb9f1509250 [0x10b29b7b0]>
				{
					valid = Yes, activities = 0xa0, repeats = Yes, order = 2001000, callout = _afterCACommitHandler (0x10b44ea99), context = <CFRunLoopObserver context 0x7fb9f15027c0>
				}

				3 : <CFRunLoopSource 0x7fb9f15095d0 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
					{
						version = 0, info = 0x7fb9f15027c0, callout = _UIApplicationHandleEventQueue (0x10b41c2db)
					}
				}

				4 : <CFRunLoopSource 0x7fb9f1400990 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f170dc90 [0x10b29b7b0]>
					{
						valid = Yes, port = 1f03, source = 0x7fb9f1400990, callout = __IOHIDEventSystemClientAvailabilityCallback (0x10d4b9a2f), context = <CFMachPort context 0x7fb9f170d6d0>
					}
				}

				5 : <CFRunLoopObserver 0x7fb9f171c850 [0x10b29b7b0]>
				{
					valid = Yes, activities = 0xa0, repeats = Yes, order = 2000000, callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv (0x10efba320), context = <CFRunLoopObserver context 0x0>
				}

				6 : <CFRunLoopObserver 0x7fb9f141aba0 [0x10b29b7b0]>
				{
					valid = Yes, activities = 0x20, repeats = Yes, order = 0, callout = _UIGestureRecognizerUpdateObserver (0x10b8fb6ab), context = <CFRunLoopObserver context 0x0>
				}

				9 : <CFRunLoopObserver 0x7fb9f1509030 [0x10b29b7b0]>
				{
					valid = Yes, activities = 0xa0, repeats = Yes, order = 1999000, callout = _beforeCACommitHandler (0x10b44ea54), context = <CFRunLoopObserver context 0x7fb9f15027c0>
				}

				12 : <CFRunLoopSource 0x7fb9f1600440 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f1600160 [0x10b29b7b0]>
					{
						valid = Yes, port = 1107, source = 0x7fb9f1600440, callout = _ZL20notify_port_callbackP12__CFMachPortPvlS1_ (0x10efca8a9), context = <CFMachPort context 0x0>
					}
				}

				16 : <CFRunLoopSource 0x7fb9f16009e0 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
					{
						version = 1, info = 0x2303, callout = PurpleEventCallback (0x10e831cb0)
					}
				}

				17 : <CFRunLoopObserver 0x7fb9f1509470 [0x10b29b7b0]>
				{
					valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x10b41bc4e), context = <CFArray 0x7fb9f1509220 [0x10b29b7b0]>
					{
						type = mutable-small, count = 0, values = ()
					}
				}


				18 : <CFRunLoopSource 0x7fb9f1400a70 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = 1, context = <CFMachPort 0x7fb9f1502110 [0x10b29b7b0]>
					{
						valid = Yes, port = 1c03, source = 0x7fb9f1400a70, callout = __IOMIGMachPortPortCallback (0x10d4c1fce), context = <CFMachPort context 0x7fb9f15020a0>
					}
				}


				19 : <CFRunLoopSource 0x7fb9f149f4c0 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server>
					{
						port = 18187, subsystem = 0x10c17b920, context = 0x7fb9f171d400
					}
				}


				20 : <CFRunLoopSource 0x7fb9f1509100 [0x10b29b7b0]>
				{
					signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server>
					{
					 	port = 14855, subsystem = 0x10c168fe0, context = 0x0
				 	}
			 	}


				21 : <CFRunLoopSource 0x7fb9f140f900 [0x10b29b7b0]>
				{
					signalled = Yes, valid = Yes, order = 0, context = <CFRunLoopSource context>
					{
						version = 0, info = 0x7fb9f140f680, callout = FBSSerialQueueRunLoopSourceHandler (0x10ddeef60)
					}
				}


				22 : <CFRunLoopObserver 0x7fb9f1509390 [0x10b29b7b0]>
				{
					valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x10b41bc4e), context = <CFArray 0x7fb9f1509220 [0x10b29b7b0]>
					{
						type = mutable-small, count = 0, values = ()
					}
				}
		},


		modes = <CFBasicHash 0x7fb9f1707f50 [0x10b29b7b0]>
		{
			type = mutable set, count = 5,

			entries =>

				2 : <CFRunLoopMode 0x7fb9f17096a0 [0x10b29b7b0]>
				{
					name = UITrackingRunLoopMode, port set = 0x1807, timer port = 0x1903,

					sources0 = <CFBasicHash 0x7fb9f1709600 [0x10b29b7b0]>
					{
						type = mutable set, count = 3,

						entries =>

							0 : <CFRunLoopSource 0x7fb9f16006b0 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
								{
									version = 0, info = 0x0, callout = PurpleEventSignalCallback (0x10e82f779)
								}
							}

							1 : <CFRunLoopSource 0x7fb9f140f900 [0x10b29b7b0]>
							{
								signalled = Yes, valid = Yes, order = 0, context = <CFRunLoopSource context>
								{
									version = 0, info = 0x7fb9f140f680, callout = FBSSerialQueueRunLoopSourceHandler (0x10ddeef60)
								}
							}

							2 : <CFRunLoopSource 0x7fb9f15095d0 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
								{
									version = 0, info = 0x7fb9f15027c0, callout = _UIApplicationHandleEventQueue (0x10b41c2db)
								}
							}
					},

					sources1 = <CFBasicHash 0x7fb9f1709640 [0x10b29b7b0]>
					{
						type = mutable set, count = 7,

						entries =>

						0 : <CFRunLoopSource 0x7fb9f14008b0 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f170d9a0 [0x10b29b7b0]>
								{
									valid = Yes, port = 1e03, source = 0x7fb9f14008b0, callout = __IOHIDEventSystemClientQueueCallback (0x10d4b987e), context = <CFMachPort context 0x7fb9f170d6d0>
								}
							}

							1 : <CFRunLoopSource 0x7fb9f149f4c0 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server>
								{
									port = 18187, subsystem = 0x10c17b920, context = 0x7fb9f171d400
								}
							}

							4 : <CFRunLoopSource 0x7fb9f1600440 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f1600160 [0x10b29b7b0]>
								{
									valid = Yes, port = 1107, source = 0x7fb9f1600440, callout = _ZL20notify_port_callbackP12__CFMachPortPvlS1_ (0x10efca8a9), context = <CFMachPort context 0x0>
								}
							}

							6 : <CFRunLoopSource 0x7fb9f16009e0 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
								{
									version = 1, info = 0x2303, callout = PurpleEventCallback (0x10e831cb0)
								}
							}

							8 : <CFRunLoopSource 0x7fb9f1400a70 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = 1, context = <CFMachPort 0x7fb9f1502110 [0x10b29b7b0]>
								{
									valid = Yes, port = 1c03, source = 0x7fb9f1400a70, callout = __IOMIGMachPortPortCallback (0x10d4c1fce), context = <CFMachPort context 0x7fb9f15020a0>
								}
							}

							9 : <CFRunLoopSource 0x7fb9f1400990 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f170dc90 [0x10b29b7b0]>
								{
									valid = Yes, port = 1f03, source = 0x7fb9f1400990, callout = __IOHIDEventSystemClientAvailabilityCallback (0x10d4b9a2f), context = <CFMachPort context 0x7fb9f170d6d0>
								}
							}

							10 : <CFRunLoopSource 0x7fb9f1509100 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server>
								{
									port = 14855, subsystem = 0x10c168fe0, context = 0x0
								}
							}
					},

					observers = <CFArray 0x7fb9f15090d0 [0x10b29b7b0]>
					{
						type = mutable-small, count = 6, values = (

							0 : <CFRunLoopObserver 0x7fb9f1509390 [0x10b29b7b0]>
							{
								valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x10b41bc4e), context = <CFArray 0x7fb9f1509220 [0x10b29b7b0]>
								{
									type = mutable-small, count = 0, values = ()
								}
							}

							1 : <CFRunLoopObserver 0x7fb9f141aba0 [0x10b29b7b0]>
							{
								valid = Yes, activities = 0x20, repeats = Yes, order = 0, callout = _UIGestureRecognizerUpdateObserver (0x10b8fb6ab), context = <CFRunLoopObserver context 0x0>
							}

							2 : <CFRunLoopObserver 0x7fb9f1509030 [0x10b29b7b0]>
							{
								valid = Yes, activities = 0xa0, repeats = Yes, order = 1999000, callout = _beforeCACommitHandler (0x10b44ea54), context = <CFRunLoopObserver context 0x7fb9f15027c0>
							}

							3 : <CFRunLoopObserver 0x7fb9f171c850 [0x10b29b7b0]>
							{
								valid = Yes, activities = 0xa0, repeats = Yes, order = 2000000, callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv (0x10efba320), context = <CFRunLoopObserver context 0x0>
							}

							4 : <CFRunLoopObserver 0x7fb9f1509250 [0x10b29b7b0]>
							{
								valid = Yes, activities = 0xa0, repeats = Yes, order = 2001000, callout = _afterCACommitHandler (0x10b44ea99), context = <CFRunLoopObserver context 0x7fb9f15027c0>
							}

							5 : <CFRunLoopObserver 0x7fb9f1509470 [0x10b29b7b0]>
							{
								valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x10b41bc4e), context = <CFArray 0x7fb9f1509220 [0x10b29b7b0]>
								{
									type = mutable-small, count = 0, values = ()
								}
							}
						)
					},

					timers = (null),

					currently 482974032 (936875103045) / soft deadline in: 1.84467431e+10 sec (@ -1) / hard deadline in: 1.84467431e+10 sec (@ -1)
				},

				3 : <CFRunLoopMode 0x7fb9f1600870 [0x10b29b7b0]>
				{
					name = GSEventReceiveRunLoopMode, port set = 0x2103, timer port = 0x2203,

					sources0 = <CFBasicHash 0x7fb9f1600790 [0x10b29b7b0]>
					{
						type = mutable set, count = 1,

						entries =>
							0 : <CFRunLoopSource 0x7fb9f16006b0 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
								{
									version = 0, info = 0x0, callout = PurpleEventSignalCallback (0x10e82f779)
								}
							}
					},

					sources1 = <CFBasicHash 0x7fb9f1600220 [0x10b29b7b0]>
					{
						type = mutable set, count = 1,
						entries =>
							2 : <CFRunLoopSource 0x7fb9f1600b40 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
								{
									version = 1, info = 0x2303, callout = PurpleEventCallback (0x10e831cb0)
								}
							}
					},

					observers = (null),
					timers = (null),
					currently 482974032 (936876354235) / soft deadline in: 1.84467431e+10 sec (@ -1) / hard deadline in: 1.84467431e+10 sec (@ -1)
				},

				4 : <CFRunLoopMode 0x7fb9f1707cc0 [0x10b29b7b0]>
				{
					name = kCFRunLoopDefaultMode, port set = 0x1603, timer port = 0x1703,

					sources0 = <CFBasicHash 0x7fb9f1708c50 [0x10b29b7b0]>
					{
						type = mutable set, count = 3,

						entries =>
							0 : <CFRunLoopSource 0x7fb9f16006b0 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
								{
									version = 0, info = 0x0, callout = PurpleEventSignalCallback (0x10e82f779)
								}
							}

							1 : <CFRunLoopSource 0x7fb9f140f900 [0x10b29b7b0]>
							{
								signalled = Yes, valid = Yes, order = 0, context = <CFRunLoopSource context>
								{
									version = 0, info = 0x7fb9f140f680, callout = FBSSerialQueueRunLoopSourceHandler (0x10ddeef60)
								}
							}

							2 : <CFRunLoopSource 0x7fb9f15095d0 [0x10b29b7b0]>
							{
								signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
								{
									version = 0, info = 0x7fb9f15027c0, callout = _UIApplicationHandleEventQueue (0x10b41c2db)
								}
							}
					},

					sources1 = <CFBasicHash 0x7fb9f1708c90 [0x10b29b7b0]>
					{
						type = mutable set, count = 7,
							entries =>

								0 : <CFRunLoopSource 0x7fb9f14008b0 [0x10b29b7b0]>
								{
									signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f170d9a0 [0x10b29b7b0]>
									{
										valid = Yes, port = 1e03, source = 0x7fb9f14008b0, callout = __IOHIDEventSystemClientQueueCallback (0x10d4b987e), context = <CFMachPort context 0x7fb9f170d6d0>
									}
								}

								1 : <CFRunLoopSource 0x7fb9f149f4c0 [0x10b29b7b0]>
								{
									signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server>
									{
									 	port = 18187, subsystem = 0x10c17b920, context = 0x7fb9f171d400
								 	}
								}

								4 : <CFRunLoopSource 0x7fb9f1600440 [0x10b29b7b0]>
								{
									signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f1600160 [0x10b29b7b0]>
									{
										valid = Yes, port = 1107, source = 0x7fb9f1600440, callout = _ZL20notify_port_callbackP12__CFMachPortPvlS1_ (0x10efca8a9), context = <CFMachPort context 0x0>
									}
								}

								6 : <CFRunLoopSource 0x7fb9f16009e0 [0x10b29b7b0]>
								{
									signalled = No, valid = Yes, order = -1, context = <CFRunLoopSource context>
									{
										version = 1, info = 0x2303, callout = PurpleEventCallback (0x10e831cb0)
									}
								}

								8 : <CFRunLoopSource 0x7fb9f1400a70 [0x10b29b7b0]>
								{
									signalled = No, valid = Yes, order = 1, context = <CFMachPort 0x7fb9f1502110 [0x10b29b7b0]>
									{
										valid = Yes, port = 1c03, source = 0x7fb9f1400a70, callout = __IOMIGMachPortPortCallback (0x10d4c1fce), context = <CFMachPort context 0x7fb9f15020a0>
									}
								}

								9 : <CFRunLoopSource 0x7fb9f1400990 [0x10b29b7b0]>
								{
									signalled = No, valid = Yes, order = 0, context = <CFMachPort 0x7fb9f170dc90 [0x10b29b7b0]>
									{
										valid = Yes, port = 1f03, source = 0x7fb9f1400990, callout = __IOHIDEventSystemClientAvailabilityCallback (0x10d4b9a2f), context = <CFMachPort context 0x7fb9f170d6d0>
									}
								}

								10 : <CFRunLoopSource 0x7fb9f1509100 [0x10b29b7b0]>
								{
									signalled = No, valid = Yes, order = 0, context = <CFRunLoopSource MIG Server>
									{
										port = 14855, subsystem = 0x10c168fe0, context = 0x0
									}
								}
					},


					observers = <CFArray 0x7fb9f15091f0 [0x10b29b7b0]>
					{
						type = mutable-small, count = 6, values = ( 0 : <CFRunLoopObserver 0x7fb9f1509390 [0x10b29b7b0]>
						{
							valid = Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x10b41bc4e), context = <CFArray 0x7fb9f1509220 [0x10b29b7b0]>
							{
								type = mutable-small, count = 0, values = ()
							}
						}

						1 : <CFRunLoopObserver 0x7fb9f141aba0 [0x10b29b7b0]>
						{
							valid = Yes, activities = 0x20, repeats = Yes, order = 0, callout = _UIGestureRecognizerUpdateObserver (0x10b8fb6ab), context = <CFRunLoopObserver context 0x0>
						}

						2 : <CFRunLoopObserver 0x7fb9f1509030 [0x10b29b7b0]>
						{
							valid = Yes, activities = 0xa0, repeats = Yes, order = 1999000, callout = _beforeCACommitHandler (0x10b44ea54), context = <CFRunLoopObserver context 0x7fb9f15027c0>
						}

						3 : <CFRunLoopObserver 0x7fb9f171c850 [0x10b29b7b0]>
						{
							valid = Yes, activities = 0xa0, repeats = Yes, order = 2000000, callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv (0x10efba320), context = <CFRunLoopObserver context 0x0>
						}

						4 : <CFRunLoopObserver 0x7fb9f1509250 [0x10b29b7b0]>
						{
							valid = Yes, activities = 0xa0, repeats = Yes, order = 2001000, callout = _afterCACommitHandler (0x10b44ea99), context = <CFRunLoopObserver context 0x7fb9f15027c0>
						}

						5 : <CFRunLoopObserver 0x7fb9f1509470 [0x10b29b7b0]>
						{
							valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x10b41bc4e), context = <CFArray 0x7fb9f1509220 [0x10b29b7b0]>
							{
								type = mutable-small, count = 0, values = ()
							}
						}
					)},

					timers = <CFArray 0x7fb9f1710e60 [0x10b29b7b0]>
					{
						type = mutable-small, count = 1, values = (

							0 : <CFRunLoopTimer 0x7fb9f1710da0 [0x10b29b7b0]>
							{
								valid = Yes, firing = No, interval = 0, tolerance = 0, next fire date = 482974033 (1.42061698 @ 938298801176), callout = (Delayed Perform) UIApplication _accessibilitySetUpQuickSpeak (0x10a6d89f7 / 0x10b8429a6) (/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/System/Library/Frameworks/UIKit.framework/UIKit), context = <CFRunLoopTimer context 0x7fb9f17113e0>
							}
						)
					},

					currently 482974032 (936876412387) / soft deadline in: 1.42238876 sec (@ 938298801176) / hard deadline in: 1.42238872 sec (@ 938298801176)
				},

				5 : <CFRunLoopMode 0x7fb9f140fa90 [0x10b29b7b0]>
				{
					name = UIInitializationRunLoopMode, port set = 0x2e13, timer port = 0x320b,
					sources0 = <CFBasicHash 0x7fb9f140fb40 [0x10b29b7b0]>
					{
						type = mutable set, count = 1,

						entries =>

							1 : <CFRunLoopSource 0x7fb9f140f900 [0x10b29b7b0]>
							{
								signalled = Yes, valid = Yes, order = 0, context = <CFRunLoopSource context>
								{
									version = 0, info = 0x7fb9f140f680, callout = FBSSerialQueueRunLoopSourceHandler (0x10ddeef60)
								}
							}
					},

					sources1 = <CFBasicHash 0x7fb9f140fb80 [0x10b29b7b0]>
					{
						type = mutable set, count = 0,
						entries =>
					},

					observers = <CFArray 0x7fb9f171c950 [0x10b29b7b0]>
					{
						type = mutable-small, count = 1, values = (
							0 : <CFRunLoopObserver 0x7fb9f171c850 [0x10b29b7b0]>
							{
								valid = Yes, activities = 0xa0, repeats = Yes, order = 2000000, callout = _ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv (0x10efba320), context = <CFRunLoopObserver context 0x0>
							}
					)},

					timers = (null),

					currently 482974032 (936878249475) / soft deadline in: 1.84467431e+10 sec (@ -1) / hard deadline in: 1.84467431e+10 sec (@ -1)
				},

				6 : <CFRunLoopMode 0x7fb9f150af20 [0x10b29b7b0]>
				{
					name = kCFRunLoopCommonModes, port set = 0x3e03, timer port = 0x3f03,
					sources0 = (null),
					sources1 = (null),
					observers = (null),
					timers = (null),
					currently 482974032 (936878364889) / soft deadline in: 1.84467431e+10 sec (@ -1) / hard deadline in: 1.84467431e+10 sec (@ -1)
				},

		}
	}```
