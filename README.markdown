PXSimpleTableAdapter
====================

`PXSimpleTableAdapter` is a class for iOS which makes managing `UITableView`s easier.

![Demo App](https://github.com/Perspx/PXSimpleTableAdapter/raw/gh-pages/Pages/DemoApp.png)

`PXSimpleTableAdapter` is geared towards managing tables which have static or data which changes very infrequently, such as Settings.app, by wrapping rows and sections in Objective-C objects, to avoid having to use enumerations or dictionaries.

This interface is created using the following code:

    PXSimpleTableRow *firstRow = [PXSimpleTableRow rowWithTitle:@"First Row"];
    firstRow.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
    
    PXSimpleTableRow *secondRow = [PXSimpleTableRow rowWithTitle:@"Another Row"];
    secondRow.accessoryType = UITableViewCellAccessoryDisclosureIndicator;

    PXSimpleTableRow *thirdRow = [PXSimpleTableRow rowWithTitle:@"Other Row"];
    
    NSArray *rows = [NSArray arrayWithObjects:firstRow, secondRow, thirdRow, nil];
    
    PXSimpleTableSection *firstSection = [[PXSimpleTableSection alloc] initWithSectionHeaderTitle:@"Header"
                                                                               sectionFooterTitle:nil
	                                                                                         rows:rows];
    
    
    //Set up the second section
    PXSimpleTableRow *fourthRow = [PXSimpleTableRow rowWithTitle:@"A Row"];
    
    PXSimpleTableRow *fifthRow = [PXSimpleTableRow rowWithTitle:@"Another Row"];
    fifthRow.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
    
    PXSimpleTableRow *sixthRow = [PXSimpleTableRow rowWithTitle:@"Other Row"];
    sixthRow.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
	
	rows = [NSArray arrayWithObjects:fourthRow, fifthRow, sixthRow, nil]
    
    PXSimpleTableSection *secondSection = [[PXSimpleTableSection alloc] initWithSectionHeaderTitle:@"Another header"
                                                                                sectionFooterTitle:nil
																							  rows:rows];

	//Set the table sections
    NSArray *sections = [[NSArray alloc] initWithObjects:firstSection, secondSection, nil];
    self.tableAdapter.sections = sections;

    [firstSection release];
    [secondSection release];
    [sections release];


How the classes work
---------------------

The project uses three main classes:

- **`PXSimpleTableAdapter`**: This is the class which is responsible for managing a table view with data that is passed to it.
- **`PXSimpleTableSection`**: This is a wrapper for information about each section – its rows, and header/footer title (if applicable).
- **`PXSimpleTableRow`**: This is a wrapper for information about a row – its title and icon.

Although excellent for dynamic content, setting up a table view with static, pre-defined content can be tricky and rather verbose. Although I love [Fraser Speirs' technique](http://speirs.org/blog/2008/10/11/a-technique-for-using-uitableview-and-retaining-your-sanity.html) of using enumerations, using objects feels like a much cleaner method and not having to implement the `UITableViewDataSource` methods reduces code required.

What PXSimpleTableAdapter is not
--------------------------------

`PXSimpleTableAdapter` is primarily a convenience for menu-like interfaces which happen to use a table view (such as Settings.app) where implementing all the `UITableView`(`Delegate`|`DataSource`) methods is long winded (especially if you have multiple levels of these views.

However, `PXSimpleTableAdapter` is *not* a replacement for `UITableView` and its associated delegate/data source methods. The delegation pattern works much better when displaying dynamic content.


Using PXSimpleTableAdapter
--------------------------

1. Lay out a `UITableView` in Interface Builder (or code<sup>*</sup>).
2. `PXSimpleTableAdapter` has a property, `tableView` which is used to point to the table you want to have the data displayed in. Either instantiate a new `PXSimpleTableAdapter` instance in Interface Builder (using an Object from the Object Library) and set the `tableView` outlet to your just laid out table view, or do this in code in your view controller.
3. Create an array of `PXSimpleTableSection` objects which represent the sections in your table view.
4. Create an array of `PXSimpleTableRow` objects (which represent reach row in your table view) for each section and set these using the `-setRows:` method on `PXSimpleTableSection`.
5. Call `-setSections:` on `PXSimpleTableAdapter` and the table will be reloaded with your data.

The project includes a demo application to see how the control can be used in more detail.

Example code
------------

Although there is a demo project in the repository, here is a quick example of setting up some data to be displayed in the table:

    PXSimpleTableRow *firstRow = [PXSimpleTableRow rowWithTitle:@"First Row" icon:[UIImage imageNamed@"myImage"]];
    PXSimpleTableRow *secondRow = [PXSimpleTableRow rowWithTitle:@"Second Row" icon:[UIImage imageNamed@"anotherImage"]];

    PXSimpleTableSection *section1 = [[PXSimpleTableSection alloc] initWithRows:[NSArray arrayWithObjects:firstRow, secondRow, nil]];
    PXSimpleTableSection *section2 = ...

    [<#table adapter#> setSections:[NSArray arrayWithObjects:section1, section2, nil]];
    [section1 release];
    [section2 release];

Custom Cells
------------

Sometimes you may wish to use a customised cell other than a standard `UITableViewCell` with `PXSimpleTableAdapter`, for example displaying a cell with a `UISwitch` inside it (like in Settings.app). This can be achieved by subclassing `PXSimpleTableRow`, creating your own custom row classes.

Once this is done, you can implement up to 3 methods in your `PXSimpleTableRow` subclass:

1. `+newCellForRow` In this method you must return a newly created cell you want to use for the row in question. Since the method starts with "new", you *must* return a retained cell, in accordance with the Cocoa Memory Management Guidelines.
2. `-setUpContentsOfCell:` In this method you are passed a cell to be set up with the values for that row. This is implemented in `PXSimpleTableRow` and simply sets the title, icon (if applicable) and accessory type on the cell, based on the row's data. If you want to do anything other than this, implement the method in your `PXSimpleTableRow` subclass and perform custom cell setup.
3. `+cellIdentifier` This method returns an `NSString` which is the reusable identifier for the cell being used to display that row. By default it returns the class name of the Row subclass, but implementing this method in your `PXSimpleTableRow` subclass allows you to modify this. Note that for the table to function correctly, the reusable identifier associated with cells created in `+newCellForRow` and the reusable identifier returned here must be the same.

For an example of using a custom cell, have a look at the `SwitchRow` and `SwitchCell` classes in the Demo App.

Property List integration
-------------------------

`PXSimpleTableAdapter` can be set up programatically by creating arrays of Sections and Rows, or by passing in a property list, which simplifies the process.

The structure of a valid plist is as follows:

    <Section Array>: (
        <Single Section Dictionary>: {
            headerTitle = "Header title";
            footerTitle = "Footer title";
            rows = <Rows Array>: (
                <Single Row Dictionary>: {
                    title = "Row title";
                    iconName = "NameOfIconFileInBundle";
                    isDisclosureRow = YES;
                },

                ...
            );
        },
        
        ...
    )

This can then be loaded in and set up by calling `-[PXSimpleTableAdapter setUpTableFromPropertyList:]`. An example of the plist and plist loading can be found in the Demo App in the repository.

The sections array can optionally be wrapped in a dictionary, which would then be the root item in the property list. When `PXSimpleTableAdapter` looks for the sections array, if it is enclosed in a dictionary, the first array will be used as the sections array.

Other contributions
===========

Thanks go to [Daniel Tull](https://github.com/danielctull) for his contributions to the project regarding selection block handlers.

License
=======

PXSimpleTableAdapter is licensed under the New BSD License, as detailed below (adapted from OSI http://www.opensource.org/licenses/bsd-license.php):


Copyright &copy 2011, Alex Rozanski.
All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

- Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
- Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
- Neither the name of the author nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

