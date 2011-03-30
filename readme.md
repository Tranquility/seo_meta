# SEO Meta tags plugin

## Installation

Add to your project using your Bundler Gemfile:

    gem 'seo_meta'

Include in your model by calling the method `is_seo_meta` in the class declaration,
for example with `Page`:

    class Page < ActiveRecord::Base
      is_seo_meta
    end

Run the generator:

    rails generate seo_meta

Migrate the database (unless you want to read 'Migrating existing data' below first):

    rake db:migrate

## Migrating existing data

This step is only if you already implemented SEO meta tag fields in your model.

At this point you could add to the migration that is generated in `db/migrate/`
logic to migrate across your existing data and remove the columns from your model
afterward, for example with `Page`:

    class CreateSeoMeta < ActiveRecord::Migration

      def self.up
        # ... migration logic from the seo_meta generator ...

        # Grab the attributes of the records that currently exist
        existing_pages = ::Page.all.map(&:attributes)

        # Remove columns
        ::SeoMeta.attributes.each do |field|
          if ::Page.column_names.map(&:to_sym).include?(field)
            remove_column ::Page.table_name, field
          end
        end

        # Reset column information because otherwise the old columns will still exist.
        ::Page.reset_column_information

        # Re-attach seo_meta
        ::Page.module_eval do
          is_seo_meta
        end

        # Migrate data
        existing_pages.each do |page|
          ::Page.find(page['id']).update_attributes({
            ::SeoMeta.attributes.keys.inject({}) {|attributes, name|
              attributes.merge(name => translation[name.to_s])
            }
          })
        end
      end

      def self.down
        # Grab the attributes of the records that currently exist
        existing_pages = ::Page.all.map(&:attributes)

        # Add columns back to your model
        ::SeoMeta.attributes.each do |field, field_type|
          unless ::Page.column_names.map(&:to_sym).include?(field)
            add_column ::Page.table_name, field, field_type
          end
        end

        # Reset column information because otherwise the new columns won't exist yet.
        ::Page.reset_column_information

        # Migrate data
        existing_pages.each do |page|
          ::Page.find(page['id']).update_attributes({
            ::SeoMeta.attributes.keys.inject({}) {|attributes, name|
              attributes.merge(name => translation[name.to_s])
            }
          })
        end

        # ... migration logic from the seo_meta generator ...
      end

    end

Now, run:

    rake db:migrate

## Anything else?

Nope, all done!