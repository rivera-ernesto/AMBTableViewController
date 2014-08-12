
NBUTableViewController
======================

Storyboard and Prototype Cells-centric block-based UITableView controller to manage complex layouts.

[![Platform: iOS](https://img.shields.io/cocoapods/p/NBUTableViewController.svg?style=flat)](http://cocoadocs.org/docsets/NBUTableViewController/)
[![Version: 1.0.0](https://img.shields.io/cocoapods/v/NBUTableViewController.svg?style=flat)](http://cocoadocs.org/docsets/NBUTableViewController/)
[![License: Apache 2.0](https://img.shields.io/cocoapods/l/NBUTableViewController.svg?style=flat)](http://cocoadocs.org/docsets/NBUTableViewController/)
[![Dependency Status](https://www.versioneye.com/objective-c/NBUTableViewController/badge.svg?style=flat)](https://www.versioneye.com/objective-c/NBUTableViewController)
[![Build Status](http://img.shields.io/travis/CyberAgent/iOS-NBUTableViewController/master.svg?style=flat)](https://travis-ci.org/CyberAgent/iOS-NBUTableViewController)

_Developed as part of [Pecolly iOS](https://itunes.apple.com/us/app/pecolly-cooking-community/id544605228?mt=8)._

## Demo

A demo project is [included](Demo) in the repository and can also be tried online [here](https://app.io/OoRIaL).

## Features

![Screenshot 1](http://cyberagent.github.io/iOS-NBUTableViewController/images/screenshot1.png)　![Screenshot 2](http://cyberagent.github.io/iOS-NBUTableViewController/images/screenshot2.png)

 - Use Storyboards' Prototype Cells to design your cells.
 - Separate table code with [`NBUTableViewSection`](Source/NBUTableViewController.h#L149)'s.
 - Uses blocks instead of delegate calls and avoid having section code separated through multiple methods.
 - Individual hide/shown, add/remove sections and rows.
 - Support for dynamic height cells.
 - Support for special "No Content Cell"'s for empty sections.

## Installation

Add the following to your [CocoaPods](http://cocoapods.org)' [Podfile](http://guides.cocoapods.org/syntax/podfile.html):

```ruby
platform :ios, '5.0'

pod 'NBUTableViewController'
```

## Documentation

http://cocoadocs.org/docsets/NBUTableViewController/

## Sample Code

Part of the included [demo project](Demo/TableDemo/PEPhotosDetailViewController.m#L24).

### Creating and configuring sections

A section with a single "static" cell of custom height:

```obj-c
footerSection = [NBUTableViewSection
                 sectionWithObjects:@[[NBUCellIdentifier identifierFromString:@"footer"]]
                 sectionUpdateBlock:NULL
                 cellHeightBlock:^CGFloat(id object, NSIndexPath * indexPath) { return 120.0; }
                 cellIdentifierBlock:NULL
                 cellConfigurationBlock:NULL];
```

A section with a single "static" cell hidden when post is `nil`:

```obj-c
writeSection = [NBUTableViewSection
                sectionWithObjects:@[[NBUCellIdentifier identifierFromString:@"write_comment"]]
                sectionUpdateBlock:^(NBUTableViewSection * section)
                {
                    section.hidden = (weakSelf.post == nil);
                }
                cellHeightBlock:NULL
                cellIdentifierBlock:NULL
                cellConfigurationBlock:NULL];
```

A section with a single row of one of two kinds:

```obj-c
authorSection = [NBUTableViewSection
                 sectionWithObjects:@[@"author_cell"]
                 sectionUpdateBlock:^(NBUTableViewSection * section)
                 {
                     [section reloadObjectAtIndex:0];
                 }
                 cellHeightBlock:NULL
                 cellIdentifierBlock:^NSString *(id object, NSIndexPath *indexPath)
                 {
                     BOOL ownPost = [weakSelf.post.authorName isEqualToString:@"Me"];
                     return ownPost ? @"author_self" : @"author_other";
                 }
                 cellConfigurationBlock:^(id object,
                                          UITableViewCell * cell,
                                          NSIndexPath * indexPath)
                 {
                     PEPhotosDetailAuthorCell * authorCell = (PEPhotosDetailAuthorCell *)cell;
                     authorCell.authorLabel.text = weakSelf.post.authorName;
                 }],
```

A section with hideable cells:

```obj-c
NSArray * sectionObjects = @[[NBUCellIdentifier identifierFromString:@"title"],   // 0
                             [NBUCellIdentifier identifierFromString:@"image"],   // 1
                             [NBUCellIdentifier identifierFromString:@"tags"],    // 2
                             [NBUCellIdentifier identifierFromString:@"recipe"]]; // 3
topSection = [NBUTableViewSection
              sectionWithObjects:sectionObjects
              sectionUpdateBlock:^(NBUTableViewSection *section)
              {
                  [section reloadObjectAtIndex:0];
              }
              cellHeightBlock:^CGFloat(id object,
                                       NSIndexPath * indexPath)
              {
                  switch ([sectionObjects indexOfObject:object]) // Shouldn't use indexPath.row because we hide/show rows
                  {
                      case 0:
                          return 40.0;
                      case 1:
                          return 160.0;
                      case 3:
                          return 170.0;
                      default:
                          return -1.0; // Table view's default height
                  }
              }
              cellIdentifierBlock:NULL
              cellConfigurationBlock:^(id object,
                                       UITableViewCell * cell,
                                       NSIndexPath * indexPath)
              {
                  switch ([sectionObjects indexOfObject:object]) // Shouldn't use indexPath.row because we hide/show rows
                  {
                      case 0:
                      {
                          PEPhotosDetailTitleCell * titleCell = (PEPhotosDetailTitleCell *)cell;
                          titleCell.titleLabel.text = weakSelf.post.title;
                          break;
                      }
                      case 1:
                      {
                          //PEPhotosDetailImageCell * imageCell = (PEPhotosDetailImageCell *)cell;
                          break;
                      }
                  }
              }];

// Initial state
[topSection setObjectsAtIndexes:[NSIndexSet indexSetWithIndexesInRange:NSMakeRange(2, 2)]
                         hidden:YES];
```

A section with a dynamic number of cells of dynamic height and a special "no content cell":

```obj-c
commentsSection = [NBUTableViewSection
                   sectionWithObjects:@[[NBUCellIdentifier identifierFromString:@"loading_comments"]]
                   sectionUpdateBlock:NULL
                   cellHeightBlock:^CGFloat(id object,
                                            NSIndexPath * indexPath)
                   {
                       if ([object isKindOfClass:[NBUCellIdentifier class]])
                       {
                           return -1.0; // Loading comments (default height)
                       }
                       if (!object)
                       {
                           return 88.0; // No content cell
                       }
                       
                       // Dynamic height comments
                       return [weakSelf heightForCellWithIdentifier:@"comment"
                                                               text:object
                                             limitedToNumberOfLines:0];
                   }
                   cellIdentifierBlock:^NSString *(id object,
                                                   NSIndexPath * indexPath)
                   {
                       return (object ? @"comment" : // A comment
                               @"no_comments");      // No content cell
                   }
                   cellConfigurationBlock:^(id object,
                                            UITableViewCell * cell,
                                            NSIndexPath * indexPath)
                   {
                       if ([cell isKindOfClass:[PEPhotosDetailCommentCell class]] &&
                           [object isKindOfClass:[NSString class]])
                       {
                           ((PEPhotosDetailCommentCell *)cell).bodyLabel.text = (NSString *)object;
                       }
                   }];

// Enable "no content cell"
commentsSection.presentsNoContentCell = YES;
```

### Configuring the initial table configuration

```obj-c
tableViewController.sections = @[topSection,
                                 authorSection,
                                 writeSection,
                                 commentsSection,
                                 footerSection];
```

### Updating the table

Updating all sections:

```obj-c
- (void)setPost:(PEPost *)post
{
    _post = post;
    
    [self updateAllSections];
}
```

Toggling rows:

```obj-c
- (IBAction)toggleDetails:(id)sender
{
    // Hide if all hiddeable rows are hidden, show all otherwise
    [topSection setObjectsAtIndexes:[NSIndexSet indexSetWithIndexesInRange:NSMakeRange(1, 3)]
                             hidden:(![topSection isObjectAtIndexHidden:1] &&
                                     ![topSection isObjectAtIndexHidden:2] &&
                                     ![topSection isObjectAtIndexHidden:3])];
}

```

## License

    Copyright 2014 CyberAgent Inc.

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

